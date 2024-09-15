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
interface PacketInterceptor {
    fun onPacketReceived(packet: ByteArray): ByteArray?
}
interface LanternTunnel {
  fun sendPacket(packet: ByteArray)
  fun setPacketInterceptor(interceptor: PacketInterceptor?)
  fun close()
}
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

3. Update your VpnService to implement LanternService to start forwarding packets to Lantern

4. Implement LanternVpnService in your VpnService class:

```kotlin
class MyVpnService : VpnService(), LanternVpnService {

    private lateinit var lanternTunnel: LanternTunnel

    override fun onCreate() {
        super.onCreate()
        // set up the tunnel and start forwarding packets
        lanternTunnel = Lantern.createTunnel()
        // Add IP ranges, domains, or apps to be redirected
        lanternTunnel.addIpRangeToTunnel("192.168.1.0/24")
        lanternTunnel.addDomainToTunnel("example.com")
        lanternTunnel.addAppToTunnel("com.example.app")
        // start forwarding packets
        lanternTunnel.startForwarding(this)
        // Set an interceptor to inspect/modify packets before forwarding
        partnerTunnel.setPacketInterceptor(object : PacketInterceptor {
            override fun onPacketReceived(packet: ByteArray): ByteArray? {
                // Inspect or modify the packet here
                // Return null to drop the packet
                Log.i("MyVpnService", "Received packet of size: ${packet.size}")

                // Logic to inspect and potentially drop packets
                if (shouldDropPacket(packet)) {
                    return null
                }

                // Return the packet to forward it to the local interface
                return packet
            }
        })
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
         // Capture packets from the VPN service
         val packet = capturePacket()

         // Send packet to Lantern's infrastructure
         lanternTunnel.sendPacket(packet)
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
