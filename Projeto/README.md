# Configuracao de Infraestrutura de Rede com Vagrant e VirtualBox

Este projeto cria uma infraestrutura de rede simulada utilizando **Vagrant** e **VirtualBox**. Ele provisiona tres maquinas virtuais:

1. **Servidor DHCP e DNS**.
2. **Servidor Web, FTP e NFS**.
3. **Maquina Cliente**.

## Pre-requisitos

Antes de executar o projeto, certifique-se de ter as seguintes ferramentas instaladas:

1. [Vagrant](https://www.vagrantup.com/downloads) (versao 2 ou superior)
2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
3. Conexao com a internet (para download de pacotes e boxes)

### Verificacao das instalacoes

Verifique em sua maquina fisica se o virtualbox foi intalado corretamente.
Execute os comandos abaixo no terminal para verificar:

```bash
vagrant --version
```

Se os comandos retornarem versoes validos, a ferramentas esta corretamente instalada.

---

## Estrutura do Projeto

- **`Vagrantfile`**: Arquivo principal com as configuracoes de rede e provisionamento.
- **Maquinas Virtuais**:
  - **DHCP/DNS Server**: Configura DHCP e DNS (Bind9).
  - **Web Server**: Configura Apache (Web), FTP (vsftpd) e NFS.
  - **Client**: Maquina cliente com testes DNS, FTP e NFS.

---

## Passo a Passo para Executar o Projeto

### 1. Clone o Repositorio

Baixe o projeto para sua maquina local:

```bash
git clone https://github.com/seu-usuario/nome-do-repositorio.git
cd nome-do-repositorio
```

### 2. Inicie o Ambiente Vagrant

Inicialize as maquinas virtuais utilizando o comando:

```bash
vagrant up
```

Este comando pode levar alguns minutos, pois faz o download das boxes Ubuntu e instala/configura todos os servicos.

### Configurações adicionais e ajustes

### 3. Acesse as Maquinas Virtual SERVIDOR DHCP E DNS

- **Servidor DHCP/DNS**:

  ```bash
  vagrant ssh dhcp
  ```

  Abra os arquivos **/etc/bind/db.rede.local** e **/etc/bind/db.192.168.10**:

  Comandos para acessar os arquivos:

  ```bash
  # acessar o primeiro arquivo
  sudo nano /etc/bind/db.rede.local

  # acessar o segundo arquivo
  sudo nano /etc/bind/db.192.168.10
  ```

  Verifique se na primeira linha destes dois arquvios esta da seguinte forma:

  ```bash
  $TTL 604800
  ```

  Se estiver faltando o $TTL adicione manualmente da forma que foi demostrado e salve o arquivo (Ctrl + O, Ctrl + x).

  Reinicie o serviço bind9 para aplicar as alterações:

  ```bash
  sudo systemctl restart bind9
  ```

### 4. Acesse as Maquinas Virtual Cliente

- **Maquina Clietne**:

    ```bash
    vagrant ssh client
    ```

    recomenendo acessar a maquina clinete através do proprio virtualbox para ter uma melhor visualização via interface.
    usuario: vagrant
    senha: vagrant

    Após iniciar a maquina cliente verifique se o arquivo **etc/resolv.conf** esta da seguinte forma:

    ```bash
    nameserver 192.168.10.10
    search rede.local
    ```

    Se não estiver altere o arquivo da forma que foi representada e salve as alterações do arquivo (Ctrl + O, Ctrl + x).

---

## Testes de Verificacao

Após iniciar o ambiente e executar os passoas anteriores, realize os seguintes testes para verificar o funcionamento dos servicos.

### 1. Teste do Servidor DHCP

Na maquina **Cliente**, verifique se o IP foi atribuido corretamente via DHCP:

```bash
ip a
```

O IP da placa enp0s8 deve estar no range configurado (192.168.10.11 a 192.168.10.100).

---

### 2. Teste do Servidor DNS

Na maquina **Cliente**, teste a resolucao de nomes:

```bash
nslookup services.rede.local
```

**Saida esperada**:

```plaintext
Server: 192.168.10.10
Address: 192.168.10.10#53

Name: services.rede.local
Address: 192.168.10.10
```

```bash
nslookup www.rede.local
```

**Saida esperada**:

```plaintext
Server: 192.168.10.3
Address: 192.168.10.10#53

Name: services.rede.local
Address: 192.168.10.10
```

---

### 3. Teste do Servidor Web

Na maquina **Cliente**, abra o navegador **Firefox** ou execute o comando:

```bash
curl http://www.rede.local
```

**Saida esperada**:

```html
<html>
  <head>
    <title>Servidor Web</title>
  </head>
  <body>
    <h1>Bem-vindo ao Servidor Web Apache</h1>
    <p>Esta e uma pagina de teste do servidor Apache.</p>
  </body>
</html>
```

---

### 4. Teste do Servidor FTP

Na maquina **Servidor Web**, crie um arquivo no diretorio **/home/fhpuser/teste.txt**:

```bash
sudo echo "Este é um teste de FTP" > /home/ftpuser/ftp/teste.txt
```

Na maquina **Cliente**, acesse o servidor FTP:

```bash
ftp ftp.rede.local
```

- Faca login com as credenciais:

  - **Usuario**: ftpuser
  - **Senha**: ftp123

- Realize os comandos abaixo para testar:
  ```bash
  ls                    # Listar diretorios
  get teste.txt         # Fazer download do arquivo
  bye                   # Sair do FTP
  ```
- verifique o conteudo do arquivo baixado
  ```bash
  cat teste.txt
  ```
  **Saida esperada**:
  ```bash
  Este é um teste de FTP
  ```

---

### 5. Teste do Servidor NFS

Na maquina **Cliente**, monte o compartilhamento NFS e teste:

```bash
sudo mkdir -p /mnt/nfs_share
sudo mount -t nfs 192.168.10.3:/srv/nfs_share /mnt/nfs_share
```

Crie um arquivo no compartilhamento:

```bash
sudo touch /mnt/nfs_share/teste_nfs.txt
ls /mnt/nfs_share
```

**Saida esperada**:

```plaintext
teste_nfs.txt
```

---

## Finalizacao

Para desligar as maquinas virtuais:

```bash
vagrant halt
```

Para destruir completamente o ambiente:

```bash
vagrant destroy
```

---

## Consideracoes Finais

Este é um arquivo que segue o passo a passo de execução do projeto. algumas partes como o primeiro passo, foi configurado para adicionar os comandos automaticamente mas por algum motivo que não descobri qual, não esta adicionando o **$TTL** na primeira linha dos arquivos de configuraçõa direto e reverso do DNS.

Portanto adicionei aqui nestes passos de execução para não dar problemas na hora de fazer os testes de execução e assim possa funcionar corretamente.

Para mais detalhes das configurações dos servidores, disponibilizamos uma documentação destes projeto, que pode ser acessado a partir do arquivo **DocumentationProjectFinal**.

---

**Autores**: Samuel Nunes Martins, João Victor Reialdo.
