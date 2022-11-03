# KCCNCNA-2022

Wanted to document the logistics behind my KubeCon 2022 live-cluster demonstration of [Lula](https://github.com/defenseunicorns/lula). I'll consider it a success if even one person finds the process valuable or enables them to also created offline kubernetes demonstrations without worry.

## Cluster Bootstrap

I used [Rancher Desktop](https://docs.rancherdesktop.io/) do to it's innate ability to treat air gap as a first class citizen. Once kubernetes is enabled, Rancher Desktop downloads the required artifacts and stores them locally such that internet connecitivty is no longer required to setup kubernetes - even if you reset your cluster.

## Istio prep

My demonstration was requiring istio service mesh as a pre-deployed system. As such, I needed to ensure I could repeatably deploy istio to my local cluster offline. I then created a [zarf](zarf.dev) [package](./istio/zarf.yaml) for istio base and istiod helm orchestration with the required images and performed a `zarf package create` in the `istio` directory to produce the zarf package `zarf-packge-istio-package-amd64.tar.zst`.

## Demo prep

I wanted to deploy simple workloads without dependencies on other services. I used nginx with a simple [deployment.yaml](./deployment.yaml) manifest and the nginx image. The manifest contains a variable that can be modified imperatively during deployment in a format that zarf understands. Performing the `zarf package create` from the project root produced the zarf package `zarf-package-compliance-demo-amd64.tar.zst`.

## Other items
- I had previously performed a `zarf init` which downloads and caches the init package for future use.
- I built the lula binary from the repository

# Demo Execution

- Rancher Desktop
    - Reset Cluster
- `zarf init`
    - uses a local cache of the init package
- `zarf package deploy zarf-packge-istio-package-amd64.tar.zst --confirm`
- `zarf package deploy zarf-package-compliance-demo-amd64.tar.zst --confirm --set INJECT=true`
- `lula execute oscal-component.yaml`
    - This produces a passing result
- `zarf package deploy zarf-package-compliance-demo-amd64.tar.zst --confirm --set INJECT=false`
- `lula execute oscal-component.yaml`
    - This produces a failing result