
# Helm chart Outline server

Helm chart for deploy Outline server to Kubernetes

## TL;DR

### 1. Create a TLS certificate using OpenSSL and copy to Kubernetes.

```console
mkdir -p /opt/outline/data
mkdir -p $HOME/outline/cert
cd $HOME/outline/cert
```

```console
export CERTIFICATE_NAME=outline-selfsigned
export SB_CERTIFICATE_FILE="$HOME/outline/cert/${CERTIFICATE_NAME}.crt"
export SB_PRIVATE_KEY_FILE="$HOME/outline/cert/${CERTIFICATE_NAME}.key"
```

```console
declare -a openssl_req_flags=(
  -x509
  -nodes
  -days 36500
  -newkey rsa:2048
  -subj '/CN=localhost'
  -keyout "${SB_PRIVATE_KEY_FILE}"
  -out "${SB_CERTIFICATE_FILE}"
)
```

```console
openssl req "${openssl_req_flags[@]}"
```

```console
kubectl create namespace outline
```
```console
kubectl create secret tls outline-tls -n outline --key ${SB_PRIVATE_KEY_FILE} --cert ${SB_CERTIFICATE_FILE}
```

### 2. Installing the Chart

```console
git clone git@github.com:didaktikm/charts.git
helm upgrade --install outline-server outline-server --namespace outline
```

### 3. Output associated encrypted string to use in Outline Manager

```console
export SHA=$(openssl x509 -noout -fingerprint \
    -sha256 -inform pem -in ${SB_CERTIFICATE_FILE} \
    | sed "s/://g" | sed 's/.*=//')

export EXTERNAL_IP="$(curl ifconfig.me)"

echo \{\"apiUrl\":\"https://${EXTERNAL_IP}:30800/api\",\"certSha256\":\"${SHA}\"\}
```