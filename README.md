# public-trust

Public trust anchors and well-known documents, served over GitHub Pages at
`https://bhumikaai.github.io/public-trust/`.

**Everything here is public by design.** These are verification keys and discovery
metadata — the artifacts a relying party fetches anonymously to check a signature.
No private key, token, or credential belongs in this repository.

## Why it exists

Google Cloud's Security Token Service validates a federated token by fetching the
issuer's OIDC discovery document and JWKS over the public internet. A managed
cluster (GKE, EKS) publishes those itself. A local cluster — kind on a laptop —
has no public endpoint, so its discovery documents are mirrored here instead.

## Layout

```
clusters/<cluster-name>/.well-known/openid-configuration
clusters/<cluster-name>/openid/v1/jwks
```

Each cluster directory is a self-contained OIDC issuer. The issuer URL is the
directory URL:

| Cluster | Issuer |
|---------|--------|
| `prithvi-kind` | `https://bhumikaai.github.io/public-trust/clusters/prithvi-kind` |

Other kinds of public artifact (service JWKS, `security.txt`) belong in sibling
top-level directories, not under `clusters/`.

## Refreshing a cluster's JWKS

The published keys must match what the cluster's API server currently signs with.
Re-publish after anything that rotates the service-account signing key — including
recreating the cluster:

```sh
kubectl --context kind-<cluster> get --raw /openid/v1/jwks \
  > clusters/<cluster-name>/openid/v1/jwks
```

The discovery document is static and only changes if the issuer URL moves.

## Two things that will silently break this

- **`.nojekyll` must stay at the repository root.** GitHub Pages runs Jekyll by
  default, and Jekyll does not publish directories beginning with a dot — so
  `.well-known/` disappears and every token exchange fails with an unreachable
  discovery document.
- **Write access here is a trust anchor.** Anyone who can push to this repository
  can publish their own signing key and mint tokens Google will accept for the
  bound service accounts. Keep branch protection on `main` and review every change.
