# ArchLinux, Cisco Anyconnect 4.x  

>Distros Arch based  

Após uma atualização do sistema tive problemas ao tentar usar o aplicativo.  
O downgrade da libxml não é mais viável, pois alguns programas podem parar de funcionar.  
Outro detalhe, desta vez você não só precisa do pacote, você também deve usar os pacotes e. No entanto, é perigoso fazer o downgrade desses pacotes no sistema.  

### Procedimento feito para cisco-annyconect 4

* Instalação da [dependência opcional](https://community.cisco.com/t5/vpn/anyconnect-linux-4-8-authentication-failed/td-p/4127671):

```
sudo pacman -S webkit2gtk
```

* Compilação e instalação do CISCO Anyconnect:

```
mkdir -p ~/build
cd ~/build
git clone https://aur.archlinux.org/cisco-anyconnect.git 
cd cisco-anyconnect
makepkg -Cris
sudo systemctl enable --now vpnagentd.service
systemctl status vpnagentd.service
```

* Configurar Política VPN Local AnyConnect:  
>Permite downloads do cliente. Se não configurar, alguns VPNs terão esta resposta ao tentar conexão:  
>>"Automatic profile updates are disabled and the local VPN profile does not match the secure gateway VPN profile.”  
>>{"As atualizações automáticas de perfil estão desabilitadas e o perfil VPN local não corresponde ao perfil VPN do gateway seguro".}  

```
sudo sed -i '/BypassDownloader/s/true/false/' /opt/cisco/anyconnect/AnyConnectLocalPolicy.xml
```

* Adicionar categoria para o arquivo `cisco-anyconnect.desktop`:

```
mkdir -p ~/.local/share/applications
cp -a /usr/share/applications/cisco-anyconnect.desktop ~/.local/share/applications
echo -e 'Categories=Network' >> ~/.local/share/applications/cisco-anyconnect.desktop
```

* Modificar campo **Exec** do arquivo `cisco-anyconnect.desktop`:

```
sed -i '/Exec/s/cisco-anyconnect/cisco-anyconnect-mod/' ~/.local/share/applications/cisco-anyconnect.desktop
```

* Configuração das bibliotecas, sem "sujar" a aplicação:

```
mkdir -p ~/build/cisco-lib_other
cd ~/build/cisco-lib_other
wget -c https://archive.archlinux.org/packages/l/libxml2/libxml2-2.11.5-1-x86_64.pkg.tar.zst
wget -c https://archive.archlinux.org/packages/g/glib2/glib2-2.78.0-3-x86_64.pkg.tar.zst
wget -c https://archive.archlinux.org/packages/i/icu/icu-73.2-2-x86_64.pkg.tar.zst
tar --use-compress-program=unzstd -xvf libxml2-2.11.5-1-x86_64.pkg.tar.zst usr/lib
tar --use-compress-program=unzstd -xvf glib2-2.78.0-3-x86_64.pkg.tar.zst usr/lib
tar --use-compress-program=unzstd -xvf icu-73.2-2-x86_64.pkg.tar.zst usr/lib
sudo mkdir -p  /opt/cisco/lib_other
sudo cp -av usr/lib/lib* /opt/cisco/lib_other
```

* Configuração do Script cisco-anyconnect:

```
sudo cp -a /usr/bin/cisco-anyconnect /usr/bin/cisco-anyconnect-mod
sudo sed -i "/export/s/lib/lib:\/opt\/cisco\/lib_other/" /usr/bin/cisco-anyconnect-mod
```

* Reiniciar o serviço:

```
sudo systemctl stop vpnagentd.service
sudo systemctl start vpnagentd.service
```
* Local para configurar os arquivos profiles `.xml`:

```
/opt/cisco/anyconnect/profile/
```

Para usar, basta clicar no ícone do Cisco ou pela linha de comando, fazer:

```
cisco-anyconnect-mod
```

## Link AUR:

[AUR.cisco-anyconnect#comment-951821](https://aur.archlinux.org/packages/cisco-anyconnect#comment-951821)
