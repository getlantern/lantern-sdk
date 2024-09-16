# Overview
This SDK provides access to the infrastructure of the Lantern circumvention tool. It allows partners and third parties to integrate Lantern as an alternative or backup solution for tunneling traffic.

## High-Level Overview

1. The SDK exposes an API that handles routing traffic through Lantern's infrastructure. It can be configured to work with traditional VPN tunneling or an HTTP/SOCKS system proxy.
2. For each partner that integrates the SDK, Lantern generates an API key that is tied to usage data
2. In tunnel mode, the SDK works with the host VPN service to intercept packets from the TUN interface and redirect traffic to Lantern.
4. In system proxy mode, Lantern starts local HTTP or SOCKS proxies for partner apps to redirect traffic, which is then forwarded on to Lantern's infrastructure.

## API Definition

```kotlin
interface LanternSDK {
  // authenticate using API key and setup Lantern
  fun initialize(apiKey: String): Boolean
  // create tunnel
  fun createTunnel(): LanternTunnel
  // start local HTTP and/or SOCKS proxies
  fun start(
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
  // revert system-wide proxy settings
  fun unsetSystemProxy(): Boolean
}

interface LanternTunnel {
  // Specify ip range that should be redirected to Lantern's infrastructure
  fun addIpRangeToTunnel(ip: String)
  // Specify domain that should be redirected
  fun addDomainToTunnel(domain: String)
  // Specify an app whose traffic should be redirected
  fun addAppToTunnel(app: String)

  fun startForwarding(fileDescriptor: ParcelFileDescriptor)
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

## Add the Lantern SDK to your Android app

1. In your module (app-level) Gradle file, add the dependency for the Lantern library:

```groovy
dependencies {
    implementation("org.getlantern.lantern:sdk:1.3.33")
}
```

2. Initialize the SDK

Update your AndroidManifest.xml to include your API key that Lantern will reference when the SDK is initialized.

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

3. TUN mode: Update your VpnService to setup a tunnel with Lantern and start forwarding packets

```kotlin
import org.getlantern.lantern.sdk.Lantern
import org.getlantern.lantern.sdk.LanternTunnel
import org.getlantern.lantern.sdk.VpnConnectionListener

class MyVpnService : VpnService() {

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
        // Establish VPN connection
        vpnInterface = builder.establish()
        // Provide the interface to the SDK to start forwarding packets
        lanternTunnel.startForwarding(vpnInterface)
     }
}
``` 

4. System proxy mode: Start local HTTP and SOCKS proxies. You can redirect traffic via these proxies to Lantern's infrastructure.
- onSuccess: Callback function invoked after starting the local proxy. Returns a ProxyInfo object containing the addresses of the local HTTP and SOCKS proxies.
- onFailure: Callback function providing an error message when the local proxy fails to start.

```Kotlin
import org.getlantern.lantern.sdk.Lantern

fun startLantern() {
    Lantern.start(
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
