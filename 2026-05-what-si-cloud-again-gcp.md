- ![Different cloud computing models and service structures](https://www.gstatic.com/bricks/image/Zpw-v4ZOiAkbLm9ARSl68tGaZFYsFsz1ABwRbl8Cj_ozj12jCTPmgVGKBARz3Xwum1CUsMQ7Hog.jpeg){:height 710, :width 1248}
- from https://cloud.google.com/learn/paas-vs-iaas-vs-saas
- ## Context
	- The [[cloud]] service models trade off control for convenience. As you move left → right (On-Premises → SaaS), the [[cloud provider]] takes over more of the stack and you manage less.
	- The stack has 7 layers, from physical (bottom) to your data (top): Hardware → Virtualization → OS → Runtime → Scaling → Application Code → Data & Configurations.
	- Source: Priyanka Vergadia (@pvergadia), thecloudgirl.dev #GCPSketchnote, 08.11.2021
- ## Responsibility by Service Model
	- **You** = you manage; **Provider** = cloud provider manages. Layers ordered top (your data) → bottom (physical hardware).
	  
	  | Layer | On-Premises | IaaS | CaaS | PaaS | FaaS | SaaS |
	  |---|---|---|---|---|---|---|
	  | Data & Configurations | You | You | You | You | You | You |
	  | Application Code | You | You | You | You | You | Provider |
	  | Scaling | You | You | You | You | Provider | Provider |
	  | Runtime | You | You | You | Provider | Provider | Provider |
	  | OS | You | You | Provider | Provider | Provider | Provider |
	  | Virtualization | You | Provider | Provider | Provider | Provider | Provider |
	  | Hardware | You | Provider | Provider | Provider | Provider | Provider |
- ## Quick read
	- **Traditional On-Premises**: you manage everything, all 7 layers.
	- **IaaS** (Infrastructure as a Service): provider handles Hardware + Virtualization; you handle OS upward. e.g. raw VMs.
	- **CaaS** (Containers as a Service): provider adds the OS; you manage Runtime upward. e.g. managed Kubernetes.
	- **PaaS** (Platform as a Service): provider adds the Runtime; you manage Scaling, App Code, Data. e.g. App Engine.
	- **FaaS** (Function as a Service): provider adds Scaling; you manage only App Code + Data. e.g. Cloud Functions / serverless.
	- **SaaS** (Software as a Service): provider runs everything except your Data & Configurations. e.g. Gmail, Salesforce.
	- The only layer **always yours** across every model is **Data & Configurations** — your data is never the provider's responsibility to manage.