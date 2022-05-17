# api-server代码阅读

## Options

#### ServerRunOptions

```go
// k8s.io/kubernetes/cmd/kube-apiserver/app/options/options.go
// ServerRunOptions runs a kubernetes api server.
type ServerRunOptions struct {
	GenericServerRunOptions *genericoptions.ServerRunOptions
	Etcd                    *genericoptions.EtcdOptions
	SecureServing           *genericoptions.SecureServingOptionsWithLoopback
	InsecureServing         *genericoptions.DeprecatedInsecureServingOptionsWithLoopback
	Audit          *genericoptions.AuditOptions              // 审计
	Features       *genericoptions.FeatureOptions
	Admission      *kubeoptions.AdmissionOptions             // 准入
	Authentication *kubeoptions.BuiltInAuthenticationOptions // 认证
	Authorization  *kubeoptions.BuiltInAuthorizationOptions  // 授权
	CloudProvider  *kubeoptions.CloudProviderOptions         // 云供应商
	APIEnablement  *genericoptions.APIEnablementOptions
	EgressSelector *genericoptions.EgressSelectorOptions

	AllowPrivileged      bool            // 是否允许特权执行
	EnableLogsHandler    bool
	EventTTL             time.Duration
	KubeletConfig        kubeletclient.KubeletClientConfig // kubelet
	KubernetesServiceNodePort int       // NodePort端口
	MaxConnectionBytesPerSec  int64     
	// ServiceClusterIPRange is mapped to input provided by user
	ServiceClusterIPRanges string       // ?
	//PrimaryServiceClusterIPRange and SecondaryServiceClusterIPRange are the results
	// of parsing ServiceClusterIPRange into actual values
	PrimaryServiceClusterIPRange   net.IPNet  // IPV4
	SecondaryServiceClusterIPRange net.IPNet  // IPV6

	ServiceNodePortRange utilnet.PortRange    
	SSHKeyfile           string
	SSHUser              string

	ProxyClientCertFile string
	ProxyClientKeyFile  string

	EnableAggregatorRouting bool

	MasterCount            int
	EndpointReconcilerType string

	ServiceAccountSigningKeyFile     string
	ServiceAccountIssuer             serviceaccount.TokenGenerator
	ServiceAccountTokenMaxExpiration time.Duration
}
```

#### ServerRunOptions

```go
// kubernetes/staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go
// ServerRunOptions contains the options while running a generic api server.
type ServerRunOptions struct {
	AdvertiseAddress net.IP

	CorsAllowedOriginList       []string
	ExternalHost                string
	MaxRequestsInFlight         int
	MaxMutatingRequestsInFlight int
	RequestTimeout              time.Duration
	LivezGracePeriod            time.Duration
	MinRequestTimeout           int
	ShutdownDelayDuration       time.Duration
	// We intentionally did not add a flag for this option. Users of the
	// apiserver library can wire it to a flag.
	JSONPatchMaxCopyBytes int64
	// The limit on the request body size that would be accepted and
	// decoded in a write request. 0 means no limit.
	// We intentionally did not add a flag for this option. Users of the
	// apiserver library can wire it to a flag.
	MaxRequestBodyBytes       int64
	TargetRAMMB               int
	EnableInfightQuotaHandler bool
}
```

#### EtcdOptions

```go
// k8s.io/kubernetes/staging/src/k8s.io/apiserver/pkg/server/options/etcd.go
type EtcdOptions struct {
	// The value of Paging on StorageConfig will be overridden by the
	// calculated feature gate value.
	StorageConfig                    storagebackend.Config
	EncryptionProviderConfigFilepath string

	EtcdServersOverrides []string

	// To enable protobuf as storage format, it is enough
	// to set it to "application/vnd.kubernetes.protobuf".
	DefaultStorageMediaType string
	DeleteCollectionWorkers int
	EnableGarbageCollection bool

	// Set EnableWatchCache to false to disable all watch caches
	EnableWatchCache bool
	// Set DefaultWatchCacheSize to zero to disable watch caches for those resources that have no explicit cache size set
	DefaultWatchCacheSize int
	// WatchCacheSizes represents override to a given resource
	WatchCacheSizes []
```

#### SecureServingOptions

```go
// kubernetes/staging/src/k8s.io/apiserver/pkg/server/options/serving.go
type SecureServingOptions struct {
	BindAddress net.IP
	// BindPort is ignored when Listener is set, will serve https even with 0.
	BindPort int
	// BindNetwork is the type of network to bind to - defaults to "tcp", accepts "tcp",
	// "tcp4", and "tcp6".
	BindNetwork string
	// Required set to true means that BindPort cannot be zero.
	Required bool
	// ExternalAddress is the address advertised, even if BindAddress is a loopback. By default this
	// is set to BindAddress if the later no loopback, or to the first host interface address.
	ExternalAddress net.IP

	// Listener is the secure server network listener.
	// either Listener or BindAddress/BindPort/BindNetwork is set,
	// if Listener is set, use it and omit BindAddress/BindPort/BindNetwork.
	Listener net.Listener

	// ServerCert is the TLS cert info for serving secure traffic
	ServerCert GeneratableKeyCert
	// SNICertKeys are named CertKeys for serving secure traffic with SNI support.
	SNICertKeys []cliflag.NamedCertKey
	// CipherSuites is the list of allowed cipher suites for the server.
	// Values are from tls package constants (https://golang.org/pkg/crypto/tls/#pkg-constants).
	CipherSuites []string
	// MinTLSVersion is the minimum TLS version supported.
	// Values are from tls package constants (https://golang.org/pkg/crypto/tls/#pkg-constants).
	MinTLSVersion string

	// HTTP2MaxStreamsPerConnection is the limit that the api server imposes on each client.
	// A value of zero means to use the default provided by golang's HTTP/2 support.
	HTTP2MaxStreamsPerConnection int
}
```
