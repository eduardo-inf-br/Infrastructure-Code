# Gestão para Infraestrutura como código para utilizadores, grupos, diretórios e permissões em Linux

## Objetivo

Criação via terminal "bash" dos utilizadores e grupos departamentais, a criação dos diretórios de trabalho e a definição das permissões em uma máquina virtual Linux.

> Execute os comandos com uma conta com privilégios administrativos. Os exemplos destinam-se a distribuições que utilizam `groupadd` e `useradd` (por exemplo, Debian/Ubuntu). As contas criadas ficam sem palavra-passe até esta ser definida pelo administrador ou por outro processo de provisionamento.

## Estrutura pretendida

| Diretório | Grupo com acesso departamental | Utilizadores |
|---|---|---|
| `/publico` | Todos os utilizadores locais | `carlos`, `maria`, `joao_`, `debora`, `sebastiana`, `roberto`, `josefina`, `amanda`, `rogerio` |
| `/adm` | `GRP_ADM` | `carlos`, `maria`, `joao_` |
| `/ven` | `GRP_VEN` | `debora`, `sebastiana`, `roberto` |
| `/sec` | `GRP_SEC` | `josefina`, `amanda`, `rogerio` |

A associação dos utilizadores aos grupos foi lida da organização visual do primeiro diapositivo: cada conjunto de três nomes aparece sob o respetivo grupo. Todos os diretórios devem ter `root` como proprietário. Os membros de cada grupo têm acesso total ao seu diretório departamental; utilizadores que não pertençam ao grupo não têm permissões nesse diretório.

## 1. Criar os grupos

```bash
sudo groupadd GRP_ADM
sudo groupadd GRP_VEN
sudo groupadd GRP_SEC
```

Se os grupos já existirem, confirme-os com `getent group NOME_DO_GRUPO` antes de repetir os comandos.

## 2. Criar utilizadores e associá-los aos grupos

Cada conta terá o respetivo grupo departamental como grupo primário, além de diretório pessoal e shell Bash.

```bash
sudo useradd -m -s /bin/bash -g GRP_ADM carlos
sudo useradd -m -s /bin/bash -g GRP_ADM maria
sudo useradd -m -s /bin/bash -g GRP_ADM joao_

sudo useradd -m -s /bin/bash -g GRP_VEN debora
sudo useradd -m -s /bin/bash -g GRP_VEN sebastiana
sudo useradd -m -s /bin/bash -g GRP_VEN roberto

sudo useradd -m -s /bin/bash -g GRP_SEC josefina
sudo useradd -m -s /bin/bash -g GRP_SEC amanda
sudo useradd -m -s /bin/bash -g GRP_SEC rogerio
```

Para definir uma palavra-passe interativamente para cada conta, utilize `sudo passwd NOME_DE_UTILIZADOR` (por exemplo, `sudo passwd carlos`). Não inclua palavras-passe em scripts ou documentação partilhada.

## 3. Criar os diretórios e definir proprietário e grupo

```bash
sudo mkdir -p /publico /adm /ven /sec

sudo chown root:root /publico
sudo chown root:GRP_ADM /adm
sudo chown root:GRP_VEN /ven
sudo chown root:GRP_SEC /sec
```

O proprietário de todos os diretórios é `root`. O grupo proprietário dos diretórios departamentais permite atribuir acesso aos membros correspondentes.

## 4. Aplicar permissões

```bash
# Diretório partilhado: todos podem listar, criar, ler, alterar e percorrer conteúdo.
sudo chmod 0777 /publico

# Diretórios departamentais: root e o grupo correspondente têm acesso total;
# os restantes utilizadores não têm permissões. O bit setgid preserva o grupo
# do diretório nos novos itens criados no seu interior.
sudo chmod 2770 /adm /ven /sec
```

A permissão `0777` em `/publico` corresponde à permissão total indicada no enunciado. Como qualquer utilizador pode alterar o conteúdo, este diretório não deve ser usado para guardar dados confidenciais. Se for preferível impedir que um utilizador apague ou renomeie ficheiros de outros, pode usar-se `1777` (bit sticky), mas essa variante restringe parcialmente a gestão de ficheiros e não equivale a acesso irrestrito total.

O modo `2770` nos diretórios departamentais atribui `rwx` ao proprietário (`root`) e ao grupo do departamento, e nenhuma permissão a «outros». O bit setgid (`2`) faz com que novos ficheiros e subdiretórios herdem o grupo do diretório. As permissões efetivas de novos ficheiros também dependem da aplicação que os cria e da `umask` do utilizador.

## 5. Verificar a configuração

```bash
getent group GRP_ADM GRP_VEN GRP_SEC
id carlos
id debora
id josefina
stat -c '%A %a %U:%G %n' /publico /adm /ven /sec
```

Resultado esperado para os diretórios departamentais: proprietário `root`, grupo correspondente e modo `2770` (normalmente apresentado como `drwxrws---`). `/publico` deverá pertencer a `root:root` e ter modo `0777` (`drwxrwxrwx`).

Também é possível confirmar o isolamento tentando aceder a um diretório departamental com um utilizador que não pertença ao respetivo grupo. A tentativa deve ser negada, salvo se existirem permissões ACL adicionais ou outras regras de acesso configuradas no sistema.

## Resumo das permissões

| Caminho | Proprietário:grupo | Modo | Efeito |
|---|---|---:|---|
| `/publico` | `root:root` | `0777` | Todos os utilizadores têm leitura, escrita e execução/acesso ao diretório. |
| `/adm` | `root:GRP_ADM` | `2770` | `root` e membros de `GRP_ADM` têm acesso total; outros não têm acesso. |
| `/ven` | `root:GRP_VEN` | `2770` | `root` e membros de `GRP_VEN` têm acesso total; outros não têm acesso. |
| `/sec` | `root:GRP_SEC` | `2770` | `root` e membros de `GRP_SEC` têm acesso total; outros não têm acesso. |


---

**`Informática para Internet`**

O suporte fornecido por corporações permite-nos o desenvolvimento e implementação de programas, projetos e recursos para Eduardo.Inf.Br. Através dessas parcerias é possível que empreendedores e proprietários de pequenas empresas, recebam ampla variedade de conteúdos e ferramentas por meio de sistemas e suporte. "[Eduardo.Inf.Br](https://informatizar.netlify.app/)".

---

<img 
    align="left" 
    alt="HTML"
    title="HTML" 
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/html5/html5-original.svg" 
/>
<img 
    align="left" 
    alt="CSS" 
    title="CSS"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/css3/css3-original.svg" 
/>
<img 
    align="left" 
    alt="JavaScript" 
    title="JavaScript"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/javascript/javascript-original.svg" 
/>
<img 
    align="left" 
    alt="Bootstrap"
    title="Bootstrap" 
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/bootstrap/bootstrap-original.svg" 
/>
<img 
    align="left" 
    alt="Git" 
    title="Git"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/git/git-original.svg" 
/>
