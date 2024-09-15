# Overview
The Lantern SDK provides access to Lantern's infrastructure for accessing blocked apps and services. It is intended to be used by partners and third parties as an alternative route for VPN and proxy traffic, acting as a backup.

## High-Level Overview

1.
2. The SDK exposes an API that handles tunneling traffic through Lantern's infrastructure.
3. 

## API Definition

```kotlin
data class ProxyInfo(
    val httpProxyPort: Int,
    val socksProxyPort: Int
)
interface LanternSDK {
  fun initialize(apiKey: String): Boolean
  fun start(
    onSuccess: (proxyInfo: ProxyInfo) -> Unit,
    onFailure: (String) -> Unit
  )
  fun disconnect(
    onSuccess: () -> Unit,
    onFailure: (String) -> Unit
  )
  fun setSystemProxy(
    proxyInfo: ProxyInfo,
    onSuccess: () -> Unit,
    onFailure: (String) -> Unit
  )
}
```

2. Start Proxy

Start local HTTP and SOCKS proxies. You can choose to redirect traffic via these proxies.
- onSuccess: Callback function invoked after starting the local proxy. Returns a ProxyInfo object containing the addresses of the local HTTP and SOCKS proxies.
- onFailure: Callback function providing an error message when the proxy fails to start.

Below is an example of how you would integrate the SDK on Android.

