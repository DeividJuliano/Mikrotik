# Mikrotik
tutoria de mikrotik
Redes Mikrotik 


É possível acessar o mikrotik usando o win box com ip, mac, e domínio 
baixe o software atualizado do hardware com arquitetura da CPU do Router, arraste em files. O arquivo para atualizar é o routeros-mmips-6.49.13.npk
Depois de tudo atualizado, arraste os arquivos ntp, security, ipv6.

Bridge 2,3,4 coloque um ip privado, adicione essas interfaces list na bridge lan, 1 será wan, receberá DHCP ou pppoe do provedor.

IP > Addresses,Address: Informe o endereço IP (ex: 192.168.2.1/24).
Network: Se não usar CIDR, informe a máscara de sub-rede (ex: 255.255.255.0).
Interface: Selecione a interface que disponibilizará os IPs (ex: wlan1).

DHCP
IP > DHCP Server.

Na aba DHCP, clique em DHCP Setup.
Selecione a interface configurada com o IP, após clicar no botão “Next”.

Nomeie o roteador em system identity Crie um usuário e exclua o antigo permissão full.
Liberar acesso a net fazendo nat:
Ip>firewall>action>masquerade , em general selecione ether1 em out int ether(Wan), mas não marque e ok
É necessário que modem e roteador estejam na mesma faixa.

Redirecionamento de porta:
Ip>firewall>nat>+chain:dstnat, Dst.Address:(ip fixo, ou em branco para link dinamico) Protocol: 6 tcp Dst.Port:(xxxxx) clique em Action Action:dst-nat To Address: (ip do host, dvr,servidor,etc..) To Ports:(porta do host) ok

Adicionar IP:
ip address add interface=ether1 address=xxxx/x
ip address add address=xxxx/x interface=ether1
ip address print
ip route
.. (voltar)
ip route add gateway=xxxx
ip dns 8.8.8.8( dns para o mkt não para os clientes) pode usar o da operadora para os clientes gateway ou interface que chega a net.

Licença do Mikrotik
Cadastrar no site
Ir na opção>make a demo ok fazer o que se deve após o código gerado cole o no terminal 
A licença já vêm nos aparelhos 
    
system identity set name=P-edifica 
adicionando rotas
ip route add dst-address=xxxx/x gateway=xxxx (lógica rede interna do roteador vizinho ou e gateway do roteador vizinho)
  
Fazer backup 
System -> Script -> +, no name, coloque o nome do seu backup,
export file=(nome do arquivo). > run script.
para vê o backup File List>Arraste para área de trabalho e veja no bloco de notas.

backup automático
system>scheduler>+ nomeie o agendamento> on event coloque o nome usado para o agendamento, coloque nos campos interval  24:00:00, que ficara todos os dias e defina o horário desejado.

backup por E-mail (pode ser feito por script)
no hotmail, sincronizar e-mail, ativar imap e pop e os parametros colocar no Mikrotik.
tool>email>configure de acordo com as especificações ex gmail> server:64.233.186.108(google)
port:587
start tls: yes
from: metaldeivid37@hotmail.com
user: pode ser o email
password: senha do seu email
>aplicar>send email (script para email : tool e-mail send
to=metaldeivid37@hotmail.com
subject=bkp diário body=hoje
file=bkp_03122023)>run script

Para o mikrotik receber ip: ou atribua de forma estática, PPPoE
ip>dhcp cliente>+seleciona a interface que vai receber o ip(geralmente wan)deixe as outras opções, ative no sinal de certo.

para o mikrotik liberar IP:
ip>dhcpserver>dhcpsetup>escolha a interface>dhcp setup>olhe e faça como necessário,tipo o escopo da rede,dns,lease time,e ok aula 10.

Modo bridge
Bridge>+(nome se quiser)>ports>adicione a ether dentro da bridge criada(ether1 ) ok (ether2) ok. Há comunicação entre as 2 interfaces.

VPN
R1=192.168.1.0/24 ether2 matriz lan
Ether1wan ip publico 
PC Linux 192.168.1.200

R2=10.10.10.0/24 ether2 filial lan
Ether1wanxxxx ip publico
PC VPC 10.10.10.254

Objetivo lan dos routes se comunicarem
Configurando a matriz, ele será o servidor vpn, as outras redes irão se conectar nele. É necessário um ip público
R1>ppp> interface>pptpserver>enable ok>profiles>+nomeie,local address insira um ip privado(para várias filiais filiais crie uma ip pool>ip>ip pool>+escolha a faixa de ip, 10.30.10.2-10.30.10.200 nomeie e ok).continuando use a pool no remote address e ok.

>ppp>secret>+>Name e senha para sua VPN,service escolha o pptp, profile da VPN que você criou aplicar e ok.. 

R2>ppp>+seta para baixo(talvez interface se não aparecer) ache o pptp cliente>insira um nome>dial out insira um IP público da matriz usuário e senha e aplica verifica no status se conectou. Faça os testes de ping para conferir se tudo está ok. 

Agora é necessário configurar rotas, para os pcs (linux e vpc) se comunicarem
R2>ip>routes>+>Dst.Address:192.168.1.0/24(lan da onde vc quer acessar e o gateway é a vpn ‘pptpMatriz’) ok. teste para que haja o ping para a rede da matriz.

R1>ip>routes>+>Dst.Address:10.30.10.0/24(lan da onde vc que acessar e o gateway é a vpn “pptpMatriz”)

Failover com netwatch

2 links de Wan ativos quando um cair o outro assume
Winbox, logue no Mikrotik, habilite o conector romon 
Ether1 operadora1 e ether2 operadora2

Criar uma interface list, interface list,+, nomeie como links,ok
Interface list,+, adicione as ether1,ok e ether2 ok em links.

Operadora1 vai entregar ip via dhcp , ative o dhcp cliente na ether1 desmarque a rota e ok, confira se pegou ip.
Operadora2, vai entregar ip fixo pra ether2, ip address, New address (200.200.0.2/29) ok

Criar range de ip para a lan, ip address, New address 10.0.10.1/24, ether5

Rotas route list, New route, dst:0.0.0.0/0 gwt: 192.168.90.1 distance:1 ok
Comente ( operadora1) ok

Route list, New route, dst:0.0.0.0/0 gwt:200.200.0.1
distance:2 comente (operadora2)

Liberar a Net por nat
Ip, firewall, nat, New nat, action: masquerade,+, chain:srcnat, out interface list: links ok confira se não há nem um bloqueio da operadora no firewall. Teste o ping para o Google.

Monitoramento por netwatch
Tools, netwatch,+,host:8.8.8.8, intervalo:00:00:10, digite esse script em up, (/ip route enable [find comment =''operadora1'']), ok, digite esse script em down, (/ip route disable [find comment =''operadora1'']), ok
Netwatch, comente: teste ping operadora1

 netwatch
Tools, netwatch,+,host:1.1.1.1, intervalo:00:00:10, digite esse script em up, (/ip route enable [find comment =''operadora2'']), ok, digite esse script em down, (/ip route disable [find comment =''operadora2'']), ok
Netwatch, comente: teste ping operadora2. Ok

 Força o link a sair por um link
Route list, New route,dst:8.8.8.8 gwt: 192.168.90.1, comment (teste ping operadora1, ok 

Route list, New route,dst:1.1.1.1 gwt: 200.200.0.1, comment (teste ping operadora2, ok

Teste derrubando uma interface ou ativando um drop no firewall (action, drop, dst address 8.8.8.8) quando isso acontecer outro link deve assumir.

