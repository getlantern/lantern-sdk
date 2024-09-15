# Overview
The Lantern SDK provides access to the infrastructure of the Lantern circumvention tool. It is intended to be used by partners and third parties that want to use Lantern as an alternative or backup route for VPN and proxy traffic.

## High-Level Overview

1. For each partner that wants to integrate the SDK, Lantern will generate an API key that is tied to usage data to distinguish traffic among providers.
2. The SDK exposes an API that handles tunneling traffic through Lantern's infrastructure.
3. 

## API Definition

```kotlin
data class ProxyInfo(
    val httpProxyPort: Int,
    val socksProxyPort: Int
)
interface LanternSDK {
  // authenticate using API key and setup Lantern
  fun initialize(apiKey: String): Boolean
  // start local HTTP and/or SOCKS proxies
  fun start(
    onSuccess: (proxyInfo: ProxyInfo) -> Unit,
    onFailure: (String) -> Unit
  )
  // stop proxies and revert system-wide settings
  fun stop(
    onSuccess: () -> Unit,
    onFailure: (String) -> Unit
  )
  // set system-wide HTTP and SOCKS proxy settings
  fun setSystemProxy(
    proxyInfo: ProxyInfo,
    onSuccess: () -> Unit,
    onFailure: (String) -> Unit
  )
}
```

2. Start Proxy

Start local HTTP and SOCKS proxies. You can redirect traffic via these proxies to Lantern's infrastructure.
- onSuccess: Callback function invoked after starting the local proxy. Returns a ProxyInfo object containing the addresses of the local HTTP and SOCKS proxies.
- onFailure: Callback function providing an error message when the local proxy fails to start.

Below is an example of how you would integrate the SDK on Android.

4. Monitor connection
