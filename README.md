# Overview
The Lantern SDK provides access to the infrastructure of the Lantern circumvention tool. It is intended to be used by partners and third parties that want to use Lantern as an alternative or backup route for VPN and proxy traffic.

## High-Level Overview

1. Lantern will generate an API key for each partner that integrates the SDK that is tied to usage data to distinguish traffic among providers.
2. The SDK exposes an API that handles routing traffic through Lantern's infrastructure. It can be configured to work with traditional VPN tunneling along with an HTTP/SOCKS system proxy.
2. In tunnel mode, the SDK is able to redirect traffic without having to setup its own VPN connection.
4. In system proxy mode, Lantern starts local HTTP and SOCKS proxies that partner apps can choose to redirect traffic to that is forwarded to Lantern's infrastructure.

## API Definition

```kotlin
interface LanternSDK {
  // authenticate using API key and setup Lantern
  fun initialize(apiKey: String): Boolean
  // create tunnel
  fun createTunnel(): LanternTunnel
  // start local HTTP and/or SOCKS proxies
  fun startProxy(
    onSuccess: (proxyInfo: ProxyInfo) -> Unit,
    onFailure: (String) -> Unit
  )
  // stop Lantern and revert system-wide settings
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
interface LanternTunnel {
  fun addIpRangeToTunnel(ip: String)
  fun addDomainToTunnel(domain: String)
  fun addAppToTunnel(app: String)
  fun sendPacket(packet: ByteArray)
  fun setPacketInterceptor(interceptor: PacketInterceptor?)
  fun close()
}
data class ProxyInfo(
    val httpProxyPort: Int,
    val socksProxyPort: Int
)
interface PacketInterceptor {
    fun onPacketReceived(packet: ByteArray): ByteArray?
}
```

2. Start Proxy

Start local HTTP and SOCKS proxies. You can redirect traffic via these proxies to Lantern's infrastructure.
- onSuccess: Callback function invoked after starting the local proxy. Returns a ProxyInfo object containing the addresses of the local HTTP and SOCKS proxies.
- onFailure: Callback function providing an error message when the local proxy fails to start.

Below is an example of how you would integrate the SDK on Android.

4. Monitor connection


## Add the Lantern SDK to your app

1. In your module (app-level) Gradle file, add the dependency for the Lantern library for Android:

```groovy
dependencies {
    implementation("org.getlantern.lantern:sdk:1.3.33")
}
```

2. Initialize the SDK

Update your AndroidManifest.xml to include your API key that Lantern will use when the SDK is initialized.

```xml
<application
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">

    <!-- Lantern SDK -->
    <meta-data
        android:name="org.getlantern.lantern.sdk.API_KEY"
        android:value="your-api-key" />

</application>
```

3. Update your VpnService to setup a tunnel with Lantern and start forwarding packets

```kotlin
import org.getlantern.lantern.sdk.Lantern
import org.getlantern.lantern.sdk.LanternTunnel
import org.getlantern.lantern.sdk.VpnConnectionListener

class MyVpnService : VpnService(), VpnConnectionListener {

    private lateinit var lanternTunnel: LanternTunnel

    override fun onCreate() {
        super.onCreate()
        // set up the tunnel and start forwarding packets
        lanternTunnel = Lantern.createTunnel(this)
    }

    override fun onDestroy() {
        lanternTunnel.close()
        vpnInterface.close()
        super.onDestroy()
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val builder = Builder()
        builder.addAddress("10.0.0.2", 24)
        builder.addRoute("0.0.0.0", 0)  // Route all traffic through VPN

        // Establish VPN connection
        vpnInterface = builder.establish()
        // Add IP ranges, domains, or apps to be redirected
        lanternTunnel.addIpRangeToTunnel("192.168.1.0/24")
        lanternTunnel.addDomainToTunnel("example.com")
        lanternTunnel.addAppToTunnel("com.example.app")
        // Set an interceptor to inspect/modify received packets
        lanternTunnel.setPacketInterceptor(object : PacketInterceptor {
            override fun onPacketReceived(packet: ByteArray): ByteArray? {
                // Inspect or modify the packet here. Return null to drop the packet
                if (shouldDropPacket(packet)) {
                    return null
                }

                // Forward packet to the local interface
                return packet
            }
        })
        // Provide the interface to the SDK to start forwarding packets
        lanternTunnel.startForwarding(vpnInterface)
     }
}
``` 

5. Start Lantern local HTTP and SOCKS proxies

```Kotlin
import org.getlantern.lantern.sdk.Lantern

fun startLantern() {
    Lantern.startLocalProxy(
        onSuccess = { proxyInfo ->
            Toast.makeText(this, "Proxies started on HTTP:${proxyInfo.httpProxyPort} and SOCKS:${proxyInfo.socksProxyPort}", Toast.LENGTH_LONG).show()
            
            // Optionally set system proxy
            Lantern.setSystemProxy(proxyInfo)
        },
        onFailure = { error ->
            Toast.makeText(this, "Failed to start Lantern proxy: $error", Toast.LENGTH_SHORT).show()
        }
    )
}
```
