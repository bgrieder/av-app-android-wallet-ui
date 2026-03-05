# TLS Bypass and Signature Verification

This project contains a TLS bypass mechanism intended 
for testing the wallet in a local emulator together with the ewQwe Relying Party Demo Webapp, 
which is started in HTTPS mode but using self-signed certificates.
(see the ewQwe Relying Party Demo Webapp documentation for more info)

## TLS Bypass

The application includes a custom Ktor `HttpClient` configuration that trusts all certificates and skips hostname verification. This is useful when connecting to local or staging servers with self-signed certificates.

### Implementation Details

The bypass is implemented using a custom `X509TrustManager` in `NetworkModule.kt` that does not perform any checks:

```kotlin
val trustAllCerts = arrayOf<TrustManager>(
    object : X509TrustManager {
        override fun checkClientTrusted(chain: Array<out X509Certificate>?, authType: String?) {}
        override fun checkServerTrusted(chain: Array<out X509Certificate>?, authType: String?) {}
        override fun getAcceptedIssuers(): Array<X509Certificate> = arrayOf()
    }
)
```

The `HttpClient` is then configured to use this trust manager:

```kotlin
return HttpClient(Android) {
    engine {
        sslManager = { connection ->
            val sslContext = SSLContext.getInstance("TLS")
            sslContext.init(null, trustAllCerts, SecureRandom())
            connection.sslSocketFactory = sslContext.socketFactory
            connection.hostnameVerifier = HostnameVerifier { _, _ -> true }
        }
    }
}
```


> [!WARNING]
> These bypasses MUST NOT be included in production builds. They significantly weaken the security of the application by making it vulnerable to Man-in-the-Middle (MitM) attacks and data tampering.
