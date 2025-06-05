<!--
Ronivaldo Domingues de Andrade - sysdocs-linux documentation - Licensed under CC BY 4.0
-->

# Instalação e Configuração do Samba para Compartilhamento de Arquivos na Rede

Esse modo do `samba` é usado para compartilhar pastas e arquivos entre Linux e outros dispositivos (Windows, Android, etc.) em uma rede local.

1. Instalar o `samba`:

	```bash
		sudo apt update
		sudo apt install samba samba-common-bin
	```

2. Criar a pasta a ser compartilhada na rede:

	```bash
		mkdir -p sua_pasta
	```

3. Editar o arquivo de configuração do `samba`:

	Abra com seu editor preferido:

	```bash
		sudo nano /etc/samba/smb.conf
	```

	Adicione ao final do arquivo as configurações e regras de compartilhamento da pasta na rede:

	```ini
		[nome_para_a_conexao]
			path = sua_pasta
			valid users = seu_usuario
			browsable = yes
			writable = yes
			create mask = 0770
			directory mask = 0770
			public = no
	```
	Onde:

	- `[nome_para_a_conexao]`:

		Define o nome do compartilhamento que será exibido na rede. Aqui, será listado como `nome_para_a_conexao` no explorador de arquivos de outro computador (Windows, Linux, etc). O nome pode ser qualquer coisa, desde que esteja entre colchetes;

	- `path = sua_pasta`:

		Caminho real no sistema de arquivos onde os arquivos do compartilhamento estão.	
		Esse diretório precisa existir e ter permissões adequadas para leitura e escrita, conforme você quer;

	- `valid users = seu_usuario`:

		Apenas o(s) usuário(s) listados aqui poderão acessar o compartilhamento. Neste caso, apenas o usuário `seu_usuario` poderá acessar a pasta. Você pode colocar vários usuários separados por vírgula, se quiser que mais de um usuário possa acessar;
		Exemplo: `valid users = usuario1,usuario2`, ou até mesmo um grupo, se quiser:
		Exemplo: `valid users = @grupo_de_usuarios`;

		> **Importante:** Esse `seu_usuario` precisa existir **tanto como usuário do sistema Linux quanto como usuário do Samba**;

	- `browsable = yes`:

		Permite que esse compartilhamento seja visível na rede, por exemplo no "Explorador de Arquivos" do Windows. Se for `no`, ele só poderá ser acessado sabendo o nome exato.

	- `writable = yes`:

		Permite que os arquivos sejam modificados (escrita, exclusão, etc) pelos usuários que têm acesso. Se for `no`, somente leitura será permitida.

	- `guest ok = yes`:
	
		Permite o acesso sem login (anônimo).
		- Isso significa que qualquer dispositivo da rede pode acessar a pasta sem precisar de usuário e senha.
		- Usado em redes domésticas ou simples.
		- Permite que usuários sem contas (usuários "guest") possam acessar o compartilhamento.
		- Se for `no`, somente usuários com contas serão permitidos acessar o compartilhamento.

	- `create mask = 0770`:

		Define as permissões dos arquivos criados nesse compartilhamento. `0770` significa:
		- Dono e grupo têm leitura, escrita e execução;
		- Outros usuários não têm nenhuma permissão.

	- `directory mask = 0770`:

		Semelhante a linha anterior, mas para as pastas. Define as permissões das pastas criadas nesse compartilhamento. `0770` significa:
		- Dono e grupo têm leitura, escrita e execução;
		- Outros usuários não têm nenhuma permissão.
	
	- `public = no`:

		Especifica que o **compartilhamento não é público**. Isso significa que somente usuários com contas serão permitidos acessar o compartilhamento. Se for `yes`, qualquer dispositivo da rede pode acessar o compartilhamento sem precisar de usuário e senha, ou seja, requer autenticação (login com nome de usuário e senha).

	Outros comando para definir permissões possíveis:

	- `force user = root`:

		Define o usuário que será usado como proprietário da pasta compartilhada.
		- Se for `root`, todos os usuários terão acesso total ao compartilhamento, incluindo a possibilidade de criar arquivos ou pastas dentro da pasta compartilhada.

	- `force group = root`:

		Define o grupo que será usado como proprietário da pasta compartilhada.
		- Se for `root`, todos os usuários terão acesso total ao compartilhamento, incluindo a possibilidade de criar arquivos ou pastas dentro da pasta compartilhada.

	**Após a configuração, salve o arquivo.**

4. Com relação ao usuário `seu_usuario` mencionado durante a configuração do `smb.conf`:

   1. Ele deve existir como um usuário do seu Sistema Operacional.
      1. Caso você não queira compartilhar pelo mesmo usuário que usa no dia a dia do sistema, mas sim criar um novo usuario voltado somante para o compartilhamento de arquivos, crie um novo usuário com o nome que desejar usando o comando:

			```bash
				sudo adduser novo_usuario
			```

	   2. Caso você queira compartilhar pelo mesmo usuário que usa no dia a dia do sistema, mas sim criar um novo grupo para o compartilhamento de arquivos, crie um novo grupo com o nome que desejar usando o comando:

			```bash
				sudo groupadd novo_grupo
			```

      	3. Caso você queira deletar o usuário ou grupo que foi mencionado acima, use o comando:

			```bash
				sudo deluser seu_usuario
				sudo delgroup novo_grupo
			```
		4. Para renomear o usuário ou grupo que foi mencionado acima, use o comando:

			```bash
				sudo usermod -l novo_nome seu_usuario
				sudo groupmod -n novo_grupo novo_nome
			```
   2. Ele deve ter permissão para acessar o compartilhamento, incluindo a possibilidade para criar arquivos ou pastas dentro do compartilhamento.

      1.  Caso você tenha criado essa pasta usando o comando `sudo mkdir ***`, ou seja, criou a pasta usando o `sudo` (root), ou o próprio `root` (`su -`), outros usuários em modo normal (não root) poderão não conseguir adicionar ou remover arquivos da pasta criada. Para isso, você precisa dar permissão de acesso ao `seu_usuario` na pasta criada:

			```bash
				sudo chown seu_usuario sua_pasta
				sudo chown -R seu_usuario:grupo_da_pasta sua_pasta
				sudo chmod 755 sua_pasta
			```

      2. Para verificar se você já adicionou o `seu_usuario` na pasta, use o comando:

			```bash
				sudo ls -la sua_pasta
			```

   3. Então, satisfazendo os requisitos dos itens acima, vamos configurar e habilitar ele no Samba:

         1. Mesmo que o seu usuário já exista no sistema, você precisa adicioná-lo ao Samba com:

			```bash
				sudo smbpasswd -a seu_usuario
			```

      		- O comando acima irá pedir para você digitar uma senha, que será usada como senha do usuário no Samba.
      		- Isso definirá a senha para login via rede (acesso ao compartilhamento).
       
         2. Para ativar o usuário no Samba (caso desativado), use:

			```bash
				sudo smbpasswd -e seu_usuario
			```
			
         3. Para verificar se o usuário está ativo no Samba, use:

			```bash
				sudo smbpasswd -l seu_usuario
			```

   		 4. Caso queira desativá-lo no Samba, use:

			```bash
				sudo smbpasswd -d seu_usuario
			```

         5. Caso queira deletar o usuário do Samba, use:

			```bash
				sudo smbpasswd -x seu_usuario
			```

		**Obs.:** Caso você alterar o nome do usuário (`username`) no seu sistema, o Samba não irá mais reconhecê-lo, então você terá que desativar o nome antigo (item 4) e adicionar o novo nome (item 1), caso não queira manter a entrada antiga no Samba, use o item 5 para deletá-lo permanentemente.
      	

5. Reinicie o `samba`

	```bash
		sudo systemctl restart smbd
	```

6. Liberar no Firewall (se estiver usando)

	```bash
		sudo ufw allow 'samba'
	```

7. Acessar a pasta de outro PC

	```bash
		\\ip_da_sua_maquina\nome_para_a_conexao
	```
8. Em breve passos para acessar em diferentes plataformas.

> 📄 _Ronivaldo Domingues de Andrade - sysdocs-linux documentation - Licensed under CC BY 4.0_
