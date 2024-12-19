# **Instituto Federal Goiano Campus Ceres**

**Curso**: Sistema de Informação  
**Nome(a):** João Victor Reinaldo Nunes  
**Nome(a):** Samuel Nunes Martins  
**Período:** 4° 

# **Projeto Final**

O projeto implementa uma topologia de rede baseada em três máquinas virtuais (VMs), todas conectadas por uma rede privada com o servidor DHCP fornecendo IPs dinâmicos para as máquinas. Aqui está a descrição detalhada da topologia:

**Servidor DHCP**

### **Topologia de Rede**

1. **Rede Local (LAN) :**  
* **Faixa de IP :** 192.168.10.0/24.  
* É criado como uma rede privada utilizando uma interface VirtualBox ( `private_network`), permitindo a comunicação entre as máquinas virtuais na mesma rede.  
2. **Servidor Central (DHCP e DNS) :**  
* A máquina virtual definida no Vagrant atua como um servidor central com os seguintes serviços:  
  * **DHCP :**  
    * **IP fixo:** 192.168.10.10 (configurado no VirtualBox).  
    * Fornece endereços IP dinamicamente no intervalo 192.168.10.11 a 192.168.10.100 .  
    * **Configure roteadores e servidores DNS através de opções:**  
      * Gateway padrão: 192.168.10.1 .  
      * DNS primário: 192.168.10.1 .  
* **DNS (BIND9) :**  
  * Configurado como servidor autoritativo para o domínio rede.local.  
  * Responder consultas para nomes na rede local e usar encaminhadores (8.8.8.8 e 8.8.4.4) para resolver nomes fora da rede local.  
3. **Resolução de Nomes :**  
* **Resolução Direta :**  
  * Traduza nomes para IPs dentro do domínio rede.local :  
    * **`services.rede.local`→ 192.168.10.10 .**  
    * **`www.rede.local`, `ftp.rede.local`, `nfs.rede.local`→ 192.168.10.3 .**  
* **Resolução Reversa :**  
  * Traduz IPs para nomes, conforme configurado:  
    * **192.168.10.10 →** `services.rede.local`.  
    * **192.168.10.3 →** `www.rede.local`, `ftp.rede.local`, `nfs.rede.local`**.**  
4. **Encaminhamento DNS :**  
* Quando as consultas DNS não podem ser resolvidas localmente, a consulta é encaminhado para os servidores públicos do Google:  
  * 8.8.8.8 .  
  * 8.8.4.4 .

**Segmentação de sub-redes**

### **Sub-rede Configurada:**

* **Endereço da Sub-rede**: `192.168.10.0`  
* **Máscara de Sub-rede**: `255.255.255.0` (ou `/24`)  
* **Faixa de IPs atribuídos pelo DHCP**: `192.168.10.11` a `192.168.10.100`  
* **Gateway e DNS**: `192.168.10.1`

### **Detalhamento da Sub-rede:**

1. **Endereço de Rede**: O endereço da sub-rede é `192.168.10.0`, e ele é usado para identificar a rede em si. Não pode ser atribuído a nenhum dispositivo.  
2. **Máscara de Sub-rede (255.255.255.0)**: A máscara de sub-rede `255.255.255.0` (ou `/24` no formato CIDR) permite até **254 endereços utilizáveis** na sub-rede. Esses endereços vão de `192.168.10.1` a `192.168.10.254`, sendo que o primeiro é tipicamente reservado para o **gateway** (neste caso, `192.168.10.1`), e o último (`192.168.10.255`) é o **endereço de broadcast**.  
3. **Faixa DHCP**: O servidor DHCP vai atribuir endereços dentro da faixa de `192.168.10.11` a `192.168.10.100`. Esses são os endereços que o servidor DHCP pode fornecer para os dispositivos que solicitam um IP. A faixa foi configurada no arquivo `dhcpd.conf` com as seguintes linhas:

  range 192.168.10.11 192.168.10.100;

Isso significa que os dispositivos na rede podem receber qualquer IP entre `192.168.10.11` e `192.168.10.100`.

4.  **Gateway e DNS**: O servidor DHCP também configura o **gateway** e o **servidor DNS** como `192.168.10.1`:

 option routers 192.168.10.1;  
 option domain\-name\-servers 192.168.10.1;

Esses valores são passados para os clientes como parte da configuração DHCP, permitindo que eles saibam para onde enviar pacotes de fora da rede e como resolver nomes de domínio.

**Tabela de Segmentação de Sub-rede:**

| Tipo | Endereço |
| :---- | :---- |
| Endereço de rede | 192.168.10.0 |
| Máscara de Sub-rede | 255.255.255.0 |
| Endereço de Broadcast | 192.168.10.255 |
| Gateway/DNS | 192.168.10.1 |
| Faixa DHCP | 192.168.10.11 a 192.168.10.100 |
| Endereço de Broadcast | 192.168.10.255 |

### **Endereços Disponíveis:**

* A sub-rede tem 254 endereços utilizáveis, de `192.168.10.1` até `192.168.10.254`. Desses, o servidor DHCP aloca os endereços de `192.168.10.11` a `192.168.10.100`, ou seja, 90 endereços.

Os dispositivos na rede que solicitarem um IP ao servidor DHCP receberão um endereço dentro dessa faixa de `192.168.10.11` a `192.168.10.100`.

**Requisitos do Serviço a ser Implementado**

### **Requisitos de Software**

* **Sistema Operacional:**  
  * Baseado em Linux, utilizando uma distribuição como Ubuntu Server (por exemplo, `ubuntu/bionic64`).  
* **Pacotes Necessários:**  
  * **Servidor DHCP:** `isc-dhcp-server`.  
  * **Servidor DNS:** `bind9`, `bind9utils`, `bind9-doc`.

	**2\. Requisitos de Configuração**

* **Servidor DHCP:**  
  * Configuração da interface para escutar no adaptador correto (`enp0s8` no exemplo).  
  * Definição da sub-rede:  
    * **Sub-rede:** `192.168.10.0/24`.  
    * **Intervalo de IPs:** `192.168.10.11 - 192.168.10.100`.  
    * **Roteador:** `192.168.10.1`.  
    * **Servidores de DNS fornecidos aos clientes:** `192.168.10.1`.  
    * **Domínio:** `rede.local`.  
* **Servidor DNS:**  
  * Configuração de opções gerais em `named.conf.options`:  
    * Encaminhamento de consultas externas para servidores DNS públicos (`8.8.8.8` e `8.8.4.4`).  
    * Permitir consultas apenas da sub-rede `192.168.10.0/24`.  
    * Configurar o DNS como recurso.  
  * Configuração de zonas DNS em `named.conf.local`:  
    * Zona direta (`rede.local`): Define os registros DNS (A e NS) para os serviços.  
    * Zona reversa (`10.168.192.in-addr.arpa`): Define os registros PTR para resolução reversa.  
  * Arquivos de zona:  
    * **Arquivo de zona direta:** Define os registros de servidores como `services`, `www`, `ftp`, e `nfs`.  
    * **Arquivo de zona reversa:** Define os registros PTR para os respectivos endereços IP.

### **3\. Requisitos de Rede**

* **Interface de Rede:**  
  * A interface `enp0s8` deve estar configurada corretamente para o segmento `192.168.10.0/24`.  
* **Segmentação de Sub-rede:**  
  * Máscara de sub-rede: `255.255.255.0` (CIDR `/24`).  
* **Infraestrutura de Rede:**  
  * Necessidade de um roteador no endereço `192.168.10.1` para interligar a sub-rede e fornecer conectividade.

### **4\. Requisitos de Segurança**

* Configurando permissões adequadas nos arquivos de configuração do DNS e DHCP para evitar alterações não autorizadas.  
* Foi restringida consultas ao DNS apenas para dispositivos na sub-rede `192.168.10.0/24`.

### **5\. Requisitos de Implementação e Manutenção**

* **Testes:**  
  * Testar o servidor DHCP para garantir que os clientes recebem IPs corretamente.  
  * Verificar a funcionalidade do servidor DNS para resolução de nomes (direta e reversa).  
* **Serviços Ativados:**  
  * Garantir que `isc-dhcp-server` e `bind9` iniciem automaticamente no boot.  
* **Monitoramento e Logs:**  
  * Revisar logs de DHCP e DNS para identificar problemas.

  Com esses requisitos atendidos, o serviço estará configurado para fornecer IPs automaticamente via DHCP e realizar resolução de nomes usando DNS no segmento de rede `192.168.10.0/24`.

**Servidor Web**

### **Topologia de Rede**

### **Rede Privada (Interna)**

### O servidor web será configurado em uma **rede privada**, com o endereço IP estático `192.168.10.3`. Ele se conecta à rede interna, usando o VirtualBox (`virtualbox__intnet`) para definir uma rede isolada entre as máquinas virtuais, onde o **servidor web** irá interagir com o **servidor DHCP/DNS** configurado anteriormente.

### **1\. Detalhes da Configuração**

* **Servidor Web (`serverweb`)**:  
  * O servidor web é configurado com a imagem **Ubuntu 18.04** e recebe o nome de host `serverweb`.  
  * Ele utiliza um **endereço IP fixo** na rede privada: `192.168.10.3`.  
  * A rede privada está configurada como `virtualbox__intnet`, permitindo que o servidor web se comunique diretamente com outras máquinas na mesma rede interna.  
* **Rede Privada**:  
  * **Sub-rede**: `192.168.10.0/24`.  
  * **Endereço IP do servidor web**: `192.168.10.3`.  
  * **Gateway/DNS**: O servidor web usará o **servidor DHCP/DNS** (configurado previamente), com o gateway e DNS configurados para `192.168.10.1` (servidor DHCP) e resoluções de nome configuradas pela zona `rede.local`.

  ### **2\. Função do Servidor Web**

* O servidor **Apache** será instalado e executado no servidor web, permitindo que ele hospede sites ou aplicações para os clientes que se conectem à rede interna.

**Segmentação de sub-rede**

web.vm.network "private\_network", ip: "192.168.10.3", virtualbox\_\_intnet: true

### **Segmentação de Rede do Servidor Web:**

1. **Endereço IP:** O servidor web foi configurado com o endereço IP fixo `192.168.10.3`.  
2. **Máscara de Sub-rede:** O código não especifica explicitamente a máscara de sub-rede, mas como é uma configuração de rede privada no intervalo `192.168.10.x`, a máscara padrão geralmente utilizada é `255.255.255.0` (ou `/24` no formato CIDR).  
3. **Rede Virtual:** A opção `virtualbox__intnet: true` indica que a rede está configurada como uma rede interna no VirtualBox. Isso significa que o servidor web está em uma rede isolada onde não há acesso direto à rede externa (a não ser que configurado de outra forma).

### **Segmentação de Sub-rede:**

Com base no endereço IP (`192.168.10.3`), o servidor web estará em uma rede com o seguinte segmento:

* Endereço de Sub-rede: `192.168.10.0`  
* Máscara de Sub-rede: `255.255.255.0` (ou `/24` CIDR)  
* Faixa de IPs Disponíveis: `192.168.10.1` a `192.168.10.254` (excluindo o endereço de rede `192.168.10.0` e o endereço de broadcast `192.168.10.255`)

### **Detalhamento:**

* **Endereço do Servidor Web**: `192.168.10.3`  
  Esse servidor web, com o IP `192.168.10.3`, está configurado para fazer parte da sub-rede `192.168.10.0/24` e se comunica com outros dispositivos dentro dessa faixa de rede, como o servidor DHCP ou outros servidores na mesma rede interna, sem acesso direto à rede externa.

**Segmento de Sub-rede** 

### **Segmento de Sub-rede do Servidor Web:**

1. **Endereço IP do Servidor Web**: `192.168.10.3`  
2. **Máscara de Sub-rede**: `255.255.255.0` (ou `/24` CIDR)  
3. **Rede**: `192.168.10.0/24`  
   Com base nisso, a **faixa de IPs utilizados** nessa sub-rede seria de `192.168.10.1` até `192.168.10.254`, mas o servidor web tem o IP fixo `192.168.10.3`, que está dentro dessa faixa.

### **Detalhamento da Sub-rede:**

* **Endereço de Rede**: `192.168.10.0`  
* **Máscara de Sub-rede**: `255.255.255.0`  
* **Endereço de Broadcast**: `192.168.10.255`  
* **Faixa de IPs**: `192.168.10.1` a `192.168.10.254`  
* **Endereço do Servidor Web**: `192.168.10.3`  
  Isso significa que o servidor web está configurado na sub-rede `192.168.10.0/24`, com o endereço IP fixo `192.168.10.3`, em uma rede privada, possivelmente isolada, configurada no VirtualBox como uma rede interna (`virtualbox__intnet: true`).

**Requisitos do Serviço a ser Implementado**

### **Requisitos de Hardware e Software:**

1. **Host Machine**:  
   * Computador com suporte a virtualização habilitado (VT-x ou AMD-V).  
   * Sistema operacional host (Windows, Linux ou macOS).  
2. **Software Necessário**:  
   * VirtualBox instalado para gerenciar máquinas virtuais.  
   * Vagrant instalado para provisionamento da máquina virtual.  
3. **Requisitos de Máquina Virtual**:  
   * Uma box do Vagrant baseada em Ubuntu Server 18.04 (bionic64).  
   * Configuração de memória mínima de 1024 MB para a VM.  
4. **Rede**:  
   * Configuração de uma interface de rede privada com o IP fixo `192.168.10.3`, que pode ser gerenciado por um servidor DHCP.  
5. **Acesso ao Repositório de Pacotes**:  
   * Conexão com a internet na máquina virtual para instalar pacotes via `apt-get`.

### **Serviços e Ferramentas:**

1. **Servidor Web**:  
   * Apache2 será instalado como servidor web.  
   * Configuração de um VirtualHost para atender a um domínio específico (exemplo: `www.rede.local`).  
2. **Diretórios e Arquivos**:  
   * Diretório `/var/www/rede.local` será criado para hospedar os arquivos do site.  
   * Um arquivo de teste HTML será colocado como página inicial.  
3. **Configuração do Apache**:  
   * Habilitação do módulo `rewrite` do Apache para suporte a redirecionamentos e URLs amigáveis.  
   * Configuração e habilitação de um VirtualHost para o domínio `rede.local`.  
4. **Gestão de Rede**:  
   * Rede privada para comunicação com outros servidores no segmento `192.168.10.0/24`.

### **Considerações Adicionais:**

* O Apache será configurado para iniciar automaticamente com o sistema.  
* Para funcionamento pleno, o domínio `www.rede.local` deve ser resolvido pelo servidor DNS configurado no mesmo segmento de rede.

**Servidor Web Apache**

### **Topologia de Rede**

### **1\. Rede Privada (Interna)**

O servidor Apache está configurado na rede privada com o endereço IP fixo `192.168.10.3`, em uma rede interna (VirtualBox **`virtualbox__intnet`**). Ele utiliza o servidor DHCP/DNS para obter configurações de rede e resolver nomes de domínio.

### **2\. Detalhes da Configuração**

#### **Servidor Web (Apache)**

* **Hostname**: O servidor tem o hostname `serverweb` (conforme o Vagrantfile).  
* **Endereço IP**: O servidor web Apache recebe o IP fixo `192.168.10.3` na rede privada.  
* **Diretório do Site**: O site do servidor é configurado no diretório `/var/www/rede.local`, com um arquivo de teste `index.html` que exibe uma mensagem simples.

  #### **VirtualHost e Domínio**

* O Apache está configurado com um **VirtualHost** para o domínio `www.rede.local`, que aponta para o diretório `/var/www/rede.local`.  
* O **arquivo de configuração do VirtualHost** foi gerado em `/etc/apache2/sites-available/rede.local.conf`, com os seguintes detalhes:  
  * **ServerName**: `www.rede.local`  
  * **DocumentRoot**: `/var/www/rede.local`

  #### **Comunicação na Rede**

* A comunicação entre o servidor Apache e os **clientes** da rede interna é realizada via **rede privada**.  
* Os **clientes** recebem IPs dinamicamente via **servidor DHCP** na faixa `192.168.10.11` a `192.168.10.100`, enquanto o servidor Apache mantém o IP fixo `192.168.10.3`.

  #### **DNS e Acesso ao Domínio**

* O **servidor DNS (Bind9)** configurado anteriormente resolve o nome `www.rede.local` para o IP `192.168.10.3` dentro da rede.

### **4\. Comunicação e Acesso ao Servidor Web**

* **Clientes** que estão na mesma rede (com IPs na faixa `192.168.10.11` a `192.168.10.100`) podem acessar o servidor web usando o nome de domínio `www.rede.local` ou diretamente através do IP `192.168.10.3`.  
* Os clientes podem testar o acesso ao servidor web, acessando a página de teste configurada no diretório `/var/www/rede.local`.

**Segmento de Sub-rede**

1. **Endereço IP do Servidor Web Apache**: `192.168.10.3`  
2. **Máscara de Sub-rede**: `255.255.255.0` (ou `/24` no formato CIDR, que é uma configuração comum em redes privadas locais)  
3. **Rede**: `192.168.10.0/24`

	**Faixa de IPs na Sub-rede:**

Com a máscara de sub-rede 255.255.255.0, a faixa de IPs válidos para a rede 192.168.10.0/24 seria:

* **Endereço de Rede**: 192.168.10.0  
* **Endereço de Broadcast**: 192.168.10.255  
* **Faixa de IPs Disponíveis**: 192.168.10.1 a 192.168.10.254 (com exceção do endereço de rede e do endereço de broadcast).

### **Sub-rede do Servidor Web Apache:**

Com o IP fixo do servidor Apache sendo 192.168.10.3, ele está na sub-rede 192.168.10.0/24, com uma faixa de endereços de 192.168.10.1 a 192.168.10.254. A máquina pode se comunicar diretamente com outros dispositivos na mesma sub-rede, como o servidor DHCP ou outros servidores, desde que estejam configurados na mesma faixa de IP.

**Requisitos do Serviço a ser Implementado**

### **Requisitos do Sistema:**

1. **Sistema Operacional:**  
   * Um sistema baseado em Linux, no caso, o Ubuntu Server 18.04 (bionic 64\) está sendo utilizado.  
2. **Hardware da VM:**  
   * Memória RAM: 1024 MB (1 GB)  
   * CPU: Capacidade mínima para rodar o Ubuntu Server e Apache de forma estável.  
3. **Rede:**  
   * Configuração de rede privada, com IP fixo (192.168.10.3) atribuído manualmente ou pelo servidor DHCP.

### **Softwares e Pacotes Necessários:**

1. **Servidor Web:**  
   * Apache2, instalado via `apt-get install apache2`.  
2. **Módulos do Apache:**  
   * Módulo `rewrite` habilitado para permitir reescrita de URLs.  
3. **Gerenciamento de VirtualHosts:**  
   * Arquivo de configuração `/etc/apache2/sites-available/rede.local.conf` para configurar o VirtualHost.  
4. **Diretórios e Permissões:**  
   * Diretório raiz para o site: `/var/www/rede.local`.  
   * Arquivo `index.html` no diretório do site com conteúdo de teste.  
   * Configurações do diretório para permitir acesso e controle apropriados (opções `Indexes`, `FollowSymLinks`, `AllowOverride All`, e `Require all granted`).  
5. **Logs do Apache:**  
   * Configurações de logs para erro (`error.log`) e acesso (`access.log`) localizados no diretório padrão do Apache.

### **Serviços Necessários:**

1. **Apache:**  
   * O serviço deve estar ativo e configurado para iniciar automaticamente com o sistema (`systemctl enable apache2`).  
2. **Rede:**  
   * IP fixo funcional no segmento 192.168.10.0/24.

### **Requisitos Adicionais:**

1. **Acesso ao Servidor:**  
   * Acesso SSH ou console para configurar e provisionar o servidor.  
2. **Domínio Local:**  
   * O nome de domínio é `www.rede.local` configurado (necessita de DNS funcional ou configuração local no `/etc/hosts` dos clientes para teste).

   Com esses requisitos atendidos, o serviço será configurado corretamente para hospedar um site simples no servidor Apache.

**Servidor FTP**

### **Topologia de Rede**

### **1\. Rede Privada (Interna)**

O servidor FTP, assim como o servidor web Apache, opera dentro de uma **rede privada** configurada em uma rede local (`192.168.10.0/24`). Ele utiliza o **servidor DHCP/DNS** para obter configurações de rede, e os clientes podem acessar o serviço FTP, desde que estejam na mesma rede interna.

### **2\. Detalhes da Configuração**

#### **Servidor FTP (vsftpd)**

* **Hostname**: O servidor FTP será identificado como `serverftp` (conforme configurado na máquina virtual).  
* **Endereço IP**: O servidor FTP terá o IP fixo `192.168.10.3` na rede privada.  
* **Diretório FTP**: O diretório `/home/ftpuser/ftp` foi configurado como raiz do FTP, com subdiretórios de uploads e permissões restritas.  
* **Modo Passivo**: O servidor FTP está configurado para operar no modo passivo (usando as portas 40000 a 50000), permitindo que clientes que se conectem à rede possam transferir arquivos corretamente.

#### **Usuário FTP**

* O script cria um usuário `ftpuser` com senha `ftp123` e define o diretório `/home/ftpuser/ftp/uploads` como o diretório onde os arquivos serão carregados.

#### **Comportamento da Rede**

* O servidor FTP está disponível para os clientes que estão na **mesma rede interna** (sub-rede `192.168.10.0/24`).  
* Os **clientes FTP** podem acessar o servidor FTP usando o IP `192.168.10.3` ou configurando o domínio `ftp.rede.local` (resolvido pelo servidor DNS configurado anteriormente).  
* A comunicação ocorre sobre as portas padrão do FTP (`21` para controle e entre `40000` e `50000` para o modo passivo).

#### **Rede Privada**

* O servidor FTP (como o servidor web) está em uma **rede privada** com o IP fixo `192.168.10.3`, na faixa de IPs `192.168.10.0/24`, fornecendo serviços de FTP para os clientes na mesma rede.  
* O servidor DHCP configura automaticamente os **clientes** na faixa de IPs `192.168.10.11` a `192.168.10.100`, permitindo que eles se conectem ao servidor FTP.

### **4\. Função do Servidor FTP**

* O **servidor FTP** permite que os **clientes FTP** (clientes da rede interna) se conectem para transferir arquivos de/para o servidor.  
* **Modo Passivo** é configurado, o que é importante para permitir que clientes atrás de firewalls ou NAT façam transferências de arquivos sem problemas.  
* O diretório `uploads` está configurado com permissões específicas, onde os clientes podem fazer upload de arquivos para o servidor.

**Segmentação de sub-rede**

### **Segmento de Sub-rede do Servidor FTP:**

* **Endereço IP do Servidor FTP:** 192.168.10.3 o servidor FTP vai ter o mesmo endereço de IP que o servidor Web porque os dois foram configurados na mesma máquina.  
* **Máscara de Sub-rede**: `255.255.255.0` ou `/24` (comum para redes privadas locais).  
* **Rede**: `192.168.10.0/24`

### **Faixa de IPs Disponíveis na Sub-rede:**

* **Endereço de Rede**: `192.168.10.0`  
* **Endereço de Broadcast**: `192.168.10.255`  
* **Faixa de IPs Usáveis**: `192.168.10.1` a `192.168.10.254`

Como o servidor FTP e o servidor web estão configurados na mesma máquina, eles acompanham o mesmo endereço IP. Portanto, se o servidor web precisar usar o IP 192.168.10.3, o servidor FTP também usará o mesmo IP, ou seja, o servidor FTP e o servidor web são acessados ​​pela mesma interface de rede da máquina, com o mesmo endereço IP (192.168. 10.3). Isso significa que, embora sejam serviços diferentes, ambos podem ser acessados ​​de forma independente, mas pela mesma rede e através do mesmo IP, cada um com sua respectiva porta (por exemplo, porta 80 para o servidor web e porta 21 para o servidor FTP ).

**Requisitos do Serviço a ser Implementado**

### **Requisitos de Hardware:**

1. **Servidor**: Um sistema que possa hospedar o serviço FTP (virtual ou físico).  
   * Memória: Pelo menos **512 MB de RAM**.  
   * Armazenamento: Espaço suficiente para arquivos que serão armazenados no diretório FTP.  
2. **Rede**:  
   * Conectividade de rede ativa com uma interface configurada.  
   * Porta **21** aberta para conexões FTP ativas.  
   * Faixa de portas (opcional) **40000-50000** aberta para conexões passivas.

### **Requisitos de Software:**

1. **Sistema Operacional**:  
   * Um sistema baseado em Linux, como **Ubuntu** ou qualquer distribuição compatível com o gerenciador de pacotes `apt`.  
   * O exemplo acima é baseado no **Ubuntu**.  
2. **Pacotes Necessários**:  
   * `vsftpd`: O servidor FTP.  
   * Utilitário básico para manipular arquivos e configurar usuários (já incluído no sistema).

### **Requisitos de Configuração:**

1. **Usuário FTP**:  
   * Deve ser criado com acesso limitado ao diretório FTP, conforme descrito no script (`ftpuser` com home `/home/ftpuser/ftp`).  
2. **Diretórios e Permissões**:  
   * Diretório principal para FTP: `/home/ftpuser/ftp`.  
   * Diretório de upload: `/home/ftpuser/ftp/uploads` com permissão de escrita para o usuário FTP.  
   * Permissões definidas como:  
     * Diretório principal: **550** (somente leitura para usuários autorizados).  
     * Diretório de upload: **750** (escrita permitida).  
3. **Configuração do `vsftpd`**:  
   * Parâmetros como:  
     * FTP passivo habilitado (`pasv_enable=YES`).  
     * Portas passivas configuradas (`pasv_min_port=40000`, `pasv_max_port=50000`).  
     * Restrições de acesso, como desabilitar logins anônimos (`anonymous_enable=NO`) e habilitar logins locais (`local_enable=YES`).  
   * Proteção por `chroot` para garantir que o usuário não acesse diretórios fora de sua pasta inicial (`chroot_local_user=YES`).  
4. **Serviços e Inicialização**:  
   * Reiniciar e habilitar o serviço `vsftpd` para garantir que ele inicialize automaticamente ao reiniciar o servidor.

### **Requisitos de Segurança:**

1. **Firewall**:  
   * Configurar o firewall para permitir acesso às portas **21** e o intervalo de portas configurado para conexões passivas (40000-50000).  
2. **Credenciais**:  
   * Garantir que o usuário FTP tenha uma senha forte e única (`ftpuser:ftp123` no exemplo deve ser substituído por algo mais seguro em produção).  
3. **Logs**:  
   * Monitorar logs de acesso em `/var/log/vsftpd.log` para identificar tentativas de acesso não autorizado.  
4. **TLS/SSL** (Opcional):  
   * Para ambientes de produção, configurar criptografia TLS para proteger os dados transmitidos.

### **Dependências Externas:**

1. **Cliente FTP**:  
   * Usuários devem ter um cliente FTP (com FileZilla, Cyberduck ou comando `ftp`) para acessar o servidor.

Se esses requisitos forem atendidos, o serviço FTP será configurado com sucesso e estará pronto para uso.

**Servidor NFS**

### **Topologia de Rede**

### **1\. Rede Privada (Interna)**

O servidor **NFS** opera dentro de uma **rede privada** com o IP configurado na faixa `192.168.10.0/24`. Ele compartilha um diretório (`/srv/nfs_share`) com os **clientes** dessa rede, permitindo que esses clientes acessem esse diretório de forma remota.

### **2\. Detalhes da Configuração do Servidor NFS**

#### **Instalação e Configuração do NFS**

O **servidor NFS** é instalado usando o comando:

systemctl restart nfs\-kernel\-server

systemctl enable nfs\-kernel\-server

#### 

#### **Teste da Configuração**

* A configuração é testada com o comando `exportfs -ra`, que aplica as alterações e exporta os diretórios compartilhados para os clientes.

#### **Criação do Diretório Compartilhado**

O diretório `/srv/nfs_share` é criado para ser compartilhado na rede:

mkdir \-p /srv/nfs\_share 

chmod 777 /srv/nfs\_share

* Este diretório é configurado com permissões amplas (777) para permitir acesso total aos clientes.

#### **Configuração no Arquivo `/etc/exports`**

O diretório `/srv/nfs_share` é adicionado ao arquivo `/etc/exports` para permitir o compartilhamento com todos os clientes na rede privada (`192.168.10.0/24`):

   echo "/srv/nfs\_share 192.168.10.0/24(rw,sync,no\_root\_squash)" \>\> /etc/exports

* `rw`: Permite leitura e escrita.  
  * `sync`: Garante que as operações de escrita sejam feitas de forma síncrona.  
  * `no_root_squash`: Impede a restrição do acesso de root no lado do cliente.

#### **Reinício do Servidor NFS**

O serviço NFS é reiniciado para aplicar as configurações:

  systemctl restart nfs\-kernel\-server

  systemctl enable nfs\-kernel\-server

#### 

#### **Teste da Configuração**

* A configuração é testada com o comando `exportfs -ra`, que aplica as alterações e exporta os diretórios compartilhados para os clientes.

### **3\. Comunicação e Acesso ao Servidor NFS**

* **Clientes NFS** na rede (`192.168.10.0/24`) podem acessar o diretório `/srv/nfs_share` de forma remota usando o protocolo NFS.  
* O compartilhamento de arquivos é feito de maneira **transparente**, permitindo que os clientes montem o diretório remoto como se fosse um diretório local.

### **4\. Função do Servidor NFS**

* O **servidor NFS** oferece uma maneira de compartilhar arquivos de forma eficiente entre sistemas Linux na mesma rede privada.  
* Os **clientes** que desejam acessar o diretório compartilhado precisam montar o diretório `/srv/nfs_share` no cliente, geralmente utilizando o comando `mount`.

**Segmentação de sub-redes**

* **Rede do Servidor NFS**: O servidor está compartilhando o diretório /srv/nfs\_share com a rede **192.168.10.0/24**.  
* **Máscara de Sub-rede**: A máscara de sub-rede é **255.255.255.0** ou **/24**, o que significa que a rede suporta até 254 endereços de host utilizáveis.  
* **Faixa de IPs Usáveis**: A faixa de IPs utilizáveis na rede **192.168.10.0/24** vai de **192.168.10.1** a **192.168.10.254**, excluindo o endereço de rede (192.168.10.0) e o endereço de broadcast (192.168.10.255).

  ### **Resumo da Segmentação de Sub-rede:**

* **Rede**: 192.168.10.0/24  
* **Máscara de Sub-rede**: 255.255.255.0 ou /24  
* **Faixa de IPs Disponíveis**: 192.168.10.1 a 192.168.10.254  
  A configuração do servidor NFS está permitindo que qualquer máquina na rede **192.168.10.0/24** acesse o compartilhamento NFS.

**Requisitos do Serviço a ser Implementado**

### **Requisitos de Software:**

1. **Sistema Operacional:**  
   * Distribuição Linux compatível, como Ubuntu Server.  
2. **Pacotes Necessários:**  
   * **nfs-kernel-server:** Para configurar e gerenciar o serviço NFS.  
   * Ferramentas básicas de gerenciamento de pacotes (como `apt-get`) para instalar e configurar o NFS.  
3. **Serviços Configurados:**  
   * Diretório para compartilhamento, configurado com permissões adequadas.  
   * Configuração do arquivo `/etc/exports` para definir os compartilhamentos e restrições de acesso.

### **Requisitos de Hardware:**

1. **Espaço em Disco:**  
   * Espaço suficiente no sistema para criar e gerenciar os diretórios compartilhados.  
2. **Memória e Processamento:**  
   * Servidor com memória e CPU suficientes para suportar a carga de trabalho do serviço NFS (mínimo recomendado: 1 GB de RAM e processador dual-core para tarefas leves).

### **Requisitos de Rede:**

1. **Segmentação de Rede:**  
   * O serviço NFS está configurado para atender a máquinas no segmento de rede `192.168.10.0/24`.  
2. **Firewall e Acessos:**  
   * Portas NFS abertas no firewall para permitir o tráfego (geralmente as portas utilizadas são dinâmicas ou atribuídas, incluindo a porta padrão do RPC \- `2049`).  
   * Garantir que dispositivos clientes tenham acesso permitido ao servidor.  
3. **Clientes Compatíveis:**  
   * Clientes que suportam NFS (Linux, Unix, ou outros sistemas compatíveis com o protocolo NFS).

### **Requisitos de Configuração:**

1. **Definição do Compartilhamento:**  
   * Diretório `/srv/nfs_share` configurado como o ponto de compartilhamento.  
   * Configuração no arquivo `/etc/exports` para permitir acesso no escopo de rede definido.  
2. **Permissões:**  
   * Diretório compartilhado com permissões adequadas (`chmod 777` no exemplo acima para acesso total).  
3. **Serviço Ativo e Persistente:**  
   * Serviço configurado para iniciar automaticamente durante a inicialização do sistema.  
4. **Sincronização e Controle de Acesso:**  
   * Configuração do acesso como `rw,sync,no_root_squash`:  
     * **rw:** Permite leitura e escrita.  
     * **sync:** Sincroniza as gravações no disco antes de responder ao cliente.  
     * **no\_root\_squash:** Permite que o root no cliente tenha privilégios equivalentes no servidor.

Esses requisitos garantem o funcionamento correto do servidor NFS para compartilhamento de arquivos em rede.

