# Auto-Configuration & Profiles

## Auto-Configuration

![[autoconfiguration-conditional-flow.svg]]

- All auto-config classes listed in `META-INF/spring/...AutoConfiguration.imports`, loaded and gated by `@Conditional*` annotations
- `@ConditionalOnClass` — is the required library actually on the classpath
- `@ConditionalOnMissingBean` — has the user already defined this bean themselves (back off if so)
- `@ConditionalOnProperty` — is a specific `application.properties` key set
- `@ConditionalOnWebApplication` — servlet-based app vs plain

**Mnemonic:** *Auto-config = a pile of `@Bean` methods gated by "is the library here?" and "did you not already do this yourself?"*

---

## Application Properties / Profiles

- `application.properties` = base config, always loaded regardless of active profile
- `application-{profile}.properties` = overlay, loaded on top when that profile is active

### Merge rule — overlay by key, not full replacement

| Key exists in | Result |
|---|---|
| Both base and profile file | Profile-specific value wins |
| Base only | Base value still applies |
| Profile file only | That value is added on top |

**Mnemonic:** *Base file is the floor, profile file is the patch — only keys the patch actually mentions get overridden.*
