# Overview
The Lantern SDK provides access to Lantern's infrastructure for accessing blocked apps and services. It is intended to be used by partners and third parties as an alternative route for VPN and proxy traffic, acting as a backup.

## High-Level Overview

1.
2. The SDK exposes an API that handles tunneling traffic through Lantern's infrastructure.
3. 

## API Definition

```kotlin
interface LanternSDK {
  fun initialize(apiKey: String): Boolean
  fun start(
    onSuccess: (proxyInfo: ProxyInfo) -> Unit,
    onFailure: (String) -> Unit
  )
}
```

Below is an example of how you would integrate the SDK on Android.

