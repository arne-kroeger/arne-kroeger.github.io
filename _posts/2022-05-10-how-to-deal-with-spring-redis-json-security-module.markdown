---
layout: single
title: 'Spring Session Redis JSON with Keycloak - A journey with the allowlist of Jackson'
date: 2022-05-10 14:22:49 +0200
categories: Spring
comments: true
tags: java spring redis keycloak jackson
---

## What happened?

I'm using [Keycloak](https://keycloak.org/) in many projects as the identity provider. I also use the keycloak-spring-boot adapter to integrate my springboot app with keycloak. I also use Redis as the session storage for projects that need a session storage that is shared between services. 

As there are special needs to access this session data from Keycloak in a Node-based service I changed the Redis serializer to GenericJackson2JsonRedisSerializer.

After starting the Spring service again the application failed in storing the session data with the following error:

~~~stacktrace
Could not read JSON: The class with org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken and name of org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"]); nested exception is com.fasterxml.jackson.databind.JsonMappingException: The class with org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken and name of org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"]) org.springframework.data.redis.serializer.SerializationException: Could not read JSON: The class with org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken and name of org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"]); nested exception is com.fasterxml.jackson.databind.JsonMappingException: The class with org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken and name of org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken is not in the allowlist. If you believe this class is safe to deserialize, please provide an explicit mapping using Jackson annotations or by providing a Mixin. If the serialization is only done by a trusted source, you can also enable default typing. See https://github.com/spring-projects/spring-security/issues/4370 for details (through reference chain: org.springframework.security.core.context.SecurityContextImpl["authentication"]) 
at org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer.deserialize(GenericJackson2JsonRedisSerializer.java:152) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer.deserialize(GenericJackson2JsonRedisSerializer.java:130) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.data.redis.core.AbstractOperations.deserializeHashValue(AbstractOperations.java:355) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.data.redis.core.AbstractOperations.deserializeHashMap(AbstractOperations.java:299) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.data.redis.core.DefaultHashOperations.entries(DefaultHashOperations.java:247) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.data.redis.core.DefaultBoundHashOperations.entries(DefaultBoundHashOperations.java:183) ~[spring-data-redis-2.4.15.jar:2.4.15] 
at org.springframework.session.data.redis.RedisIndexedSessionRepository.getSession(RedisIndexedSessionRepository.java:440) ~[spring-session-data-redis-2.4.6.jar:2.4.6] 
at org.springframework.session.data.redis.RedisIndexedSessionRepository.findById(RedisIndexedSessionRepository.java:412) ~[spring-session-data-redis-2.4.6.jar:2.4.6] 
at org.springframework.session.data.redis.RedisIndexedSessionRepository.findById(RedisIndexedSessionRepository.java:249) ~[spring-session-data-redis-2.4.6.jar:2.4.6]
~~~

## Problem

This mentioned allow list was added to Spring security with this [GitHub issue](https://github.com/spring-projects/spring-security/issues/4370)

The allowlist is meant to ensure that the classes which get serialized can be unserialized. So the classes have a explizit annotation:

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS) 
```

or is added via Jackson Mixin:

```java
mapper.addMixIn(AccessToken.class, AccessToken.class);
```

As in this case there are classes that cannot be changed directly I added Mixins. There are some special needs to specify ignored fields or JSON creator constructors as these annotations are missing in the Keycloak sources.

## Solution

The implementation looks like this:

```java
@Configuration
public class SessionConfig implements BeanClassLoaderAware  {

  private ClassLoader loader;

  @Bean
  public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
    return new GenericJackson2JsonRedisSerializer(objectMapper());
  }

  private ObjectMapper objectMapper() {
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModules(SecurityJackson2Modules.getModules(this.loader));
    mapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
    mapper.addMixIn(KeycloakAuthenticationToken.class, KeycloakAuthenticationTokenMixin.class);
    mapper.addMixIn(SimpleKeycloakAccount.class, SimpleKeycloakAccountMixin.class);
    mapper.addMixIn(KeycloakPrincipal.class, KeycloakPrincipalMixin.class);
    mapper.addMixIn(RefreshableKeycloakSecurityContext.class, RefreshableKeycloakSecurityContextMixin.class);
    mapper.addMixIn(AccessToken.class, AccessToken.class);
    mapper.addMixIn(AccessToken.Access.class, AccessToken.Access.class);
    mapper.addMixIn(IDToken.class, IDToken.class);
    mapper.addMixIn(HashSet.class, HashSet.class);

    return mapper;
  }

  @Override
  public void setBeanClassLoader(ClassLoader classLoader) {
    this.loader = classLoader;
  }

  private abstract static class KeycloakAuthenticationTokenMixin {
    @JsonCreator
    public KeycloakAuthenticationTokenMixin(@JsonProperty("details") KeycloakAccount account,
                             @JsonProperty("interactive") boolean interactive) {}

    @JsonIgnore
    public abstract Object getCredentials();

    @JsonIgnore
    public abstract OidcKeycloakAccount getAccount();

    @JsonIgnore
    public abstract String getName();
  }

  private abstract static class SimpleKeycloakAccountMixin {
    @JsonCreator
    public SimpleKeycloakAccountMixin(@JsonProperty("principal") Principal principal, @JsonProperty("roles") @JsonDeserialize(as = HashSet.class) Set<String> roles, @JsonProperty("securityContext") RefreshableKeycloakSecurityContext securityContext) {}

    @JsonIgnore
    public abstract RefreshableKeycloakSecurityContext getKeycloakSecurityContext();

  }

  private static class KeycloakPrincipalMixin <T extends KeycloakSecurityContext> {
    @JsonCreator
    public KeycloakPrincipalMixin(@JsonProperty("name") String name, @JsonProperty("keycloakSecurityContext") T context) {}
  }



  private abstract static class RefreshableKeycloakSecurityContextMixin {
    @JsonIgnore
    public abstract boolean isActive();

    @JsonIgnore
    public abstract String getRealm();

    @JsonIgnore
    public abstract KeycloakDeployment getDeployment();

  }

}
```

## Addendum

The needed changes for Keycloak has been reported in the Keyloak issue: [10483](https://github.com/keycloak/keycloak/issues/10483)

Example project / implementation can be found here: [test-spring-keycloak-redis-serialize](https://github.com/arne-kroeger/test-spring-keycloak-redis-serialize)