# Geth

## Descripción General

{% hint style="danger" %}
:octagonal\_sign:**Strongly discouraged** :octagonal\_sign:**: GETH can be** [**hazardous to your all YOUR STAKE.**](https://twitter.com/EthDreamer/status/1749355402473410714)

Seleccione un cliente minoritario.&#x20;

**Recommendación:** Besu or Nethermind.
{% endhint %}

{% hint style="info" %}
**Geth** - Go Ethereum es una de las tres implementaciones originales (junto con C++ y Python) del protocolo Ethereum. Está escrito en **Go**, es de código abierto y tiene licencia GNU LGPL v3..
{% endhint %}

#### Enlaces Oficiales

| Temas         | Enlace                                                                                                 |
| ------------- | ---------------------------------------------------------------------------------------------------- |
| Lanzamientos      | [https://github.com/ethereum/go-ethereum/releases](https://github.com/ethereum/go-ethereum/releases) |
| Documentación | [https://geth.ethereum.org/docs](https://geth.ethereum.org/docs)                                     |
| Sitio Web      | [https://geth.ethereum.org](https://geth.ethereum.org/)                                              |

### 1. Crear cuenta de servicio y directorio de datos

Cree un usuario de servicio para el servicio de ejecución, cree un directorio de datos y asigne propiedad.

```bash
sudo adduser --system --no-create-home --group execution
sudo mkdir -p /var/lib/geth
sudo chown -R execution:execution /var/lib/geth
```

### **2. Instalar binarios**

* La descarga de archivos binarios suele ser más rápida y cómoda.&#x20;
* Construir a partir del código fuente puede ofrecer una mejor compatibilidad y está más alineado con el espíritu de FOSS (software gratuito de código abierto).

<details>

<summary>Option 1 - Download binaries</summary>

<pre class="language-bash"><code class="lang-bash">RELEASE_URL="https://geth.ethereum.org/downloads"
<strong>FILE="https://gethstore.blob.core.windows.net/builds/geth-linux-amd64[a-zA-Z0-9./?=_%:-]*.tar.gz"
</strong>BINARIES_URL="$(curl -s $RELEASE_URL | grep -Eo $FILE | head -1)"

echo Downloading URL: $BINARIES_URL

cd $HOME
wget -O geth.tar.gz $BINARIES_URL
tar -xzvf geth.tar.gz -C $HOME
rm geth.tar.gz
sudo mv $HOME/geth-* geth
</code></pre>

Instalar binarios.

```bash
sudo mv $HOME/geth/geth /usr/local/bin
```

</details>

<details>

<summary>Option 2 - Build from source code</summary>

Instalar dependencias de Go

```bash
wget -O go.tar.gz https://go.dev/dl/go1.19.6.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go.tar.gz
echo export PATH=$PATH:/usr/local/go/bin >> $HOME/.bashrc
source $HOME/.bashrc
```

Verifique que Go esté instalado correctamente verificando la versión y los archivos de limpieza.

```bash
go version
rm go.tar.gz
```

Instalar dependencias de compilación.

```bash
sudo apt-get update
sudo apt install build-essential git
```

Construye el binario.

```bash
mkdir -p ~/git
cd ~/git
git clone -b master https://github.com/ethereum/go-ethereum.git
cd go-ethereum
# Get new tags
git fetch --tags
# Get latest tag name
latestTag=$(git describe --tags `git rev-list --tags --max-count=1`)
# Checkout latest tag
git checkout $latestTag
# Build
make geth
```

instalar el binario.

<pre class="language-bash"><code class="lang-bash"><strong>sudo cp $HOME/git/go-ethereum/build/bin/geth /usr/local/bin
</strong></code></pre>

</details>

    ### **3. Instalar y configurar systemd**

Cree un **archivo de unidad systemd** para definir su configuración `execution.service`.

```bash
sudo nano /etc/systemd/system/execution.service
```

Pegue la siguiente configuración en el archivo.

<pre class="language-bash"><code class="lang-bash">[Unit]
Description=Geth Execution Layer Client service for Mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=execution
Group=execution
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
ExecStart=/usr/local/bin/geth \
    --mainnet \
    --port 30303 \
    --http.port 8545 \
    --authrpc.port 8551 \
    --maxpeers 50 \
    --metrics \
    --http \
    --datadir=/var/lib/geth \
    --pprof \
    --state.scheme=path \
    --authrpc.jwtsecret=/secrets/jwtsecret
   
<strong>[Install]
</strong>WantedBy=multi-user.target
</code></pre>

Para salir y guardar, presione `Ctrl` + `X`, luego `Y`, luego `Enter`.

Ejecute lo siguiente para habilitar el inicio automático en el momento del arranque.

```bash
sudo systemctl daemon-reload
sudo systemctl enable execution
```

Finalmente, inicie su cliente de capa de ejecución y verifique su estado.

```bash
sudo systemctl start execution
sudo systemctl status execution
```

Presione `Ctrl` + `C` para salir del estado.

### 4. Comandos útiles del cliente de ejecución

{% tabs %}
{% tab title="View Logs" %}
<pre class="language-bash"><code class="lang-bash"><strong>sudo journalctl -fu execution | ccze
</strong></code></pre>

Un cliente de ejecución **Geth** que funcione correctamente indicará "Nuevo segmento de cadena potencial importado". Por ejemplo,

```
geth[4531]: INFO [02-04|01:20:48.280] Chain head was updated    number=16000 hash=2317ae..c41107
geth[4531]: INFO [02-04|01:20:49.648] Imported new potential chain segment       number=16000 hash=ab173f..33a21b
```
{% endtab %}

{% tab title="Stop" %}
```bash
sudo systemctl stop execution
```
{% endtab %}

{% tab title="Start" %}
```bash
sudo systemctl start execution
```
{% endtab %}

{% tab title="View Status" %}
```bash
sudo systemctl status execution
```
{% endtab %}

{% tab title="Reset Database" %}
Common reasons to reset the database can include:

* Recovering from a corrupted database due to power outage or hardware failure
* Re-syncing to reduce disk space usage
* Upgrading to a new storage format

```bash
sudo systemctl stop execution
sudo rm -rf /var/lib/geth/*
sudo systemctl restart execution
```

Time to re-sync the execution client can take a few hours up to a day.
{% endtab %}
{% endtabs %}

Now that your execution client is configured and started, proceed to the next step on setting up your consensus client.

{% hint style="warning" %}
If you're checking the logs and see any warnings or errors, please be patient as these will normally resolve once both your execution and consensus clients are fully synced to the Ethereum network.
{% endhint %}
