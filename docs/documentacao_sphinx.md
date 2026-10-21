# Sphinx e Git
## Documentacao de Resolucao de Problemas: Configuracao do Sphinx e Git

Este documento registra os problemas tecnicos encontrados durante a configuracao do Sphinx e a publicacao no GitHub, bem como as solucoes aplicadas para cada caso.

---

## 1. Erro ao criar o arquivo .gitignore no terminal

### Sintoma
Ao tentar configurar o arquivo `.gitignore`, o conteudo foi colado diretamente no terminal, resultando em erros como `bash: .venv/: Arquivo ou diretorio inexistente` e `bash: *.pyc: comando nao encontrado`.

### Causa
O terminal interpretou as linhas do arquivo `.gitignore` como comandos a serem executados pelo sistema, e nao como texto a ser salvo em um arquivo.

### Solucao
O arquivo `.gitignore` deve ser criado na raiz do repositorio. Para isso, e possivel utilizar o comando `cat` com um bloco de texto (heredoc) ou um editor de texto como o `nano`.

Comando utilizado para criar o arquivo de forma correta:
```bash
cat > .gitignore << 'EOF'
.venv/
__pycache__/
*.pyc
docs/_build/
EOF
```

---

## 2. Erro de permissao no Git Push (Deploy Key)

### Sintoma
Ao executar `git push origin main`, o Git retornou a mensagem:
`ERROR: Permission to wellington-cpu/Repos_de_Sala.git denied to deploy key`

### Causa
A chave SSH utilizada na maquina estava cadastrada no GitHub como uma "Deploy Key". Por padrao, essas chaves sao configuradas com permissao de leitura apenas, impedindo o envio de alteracoes (push).

### Solucao
Como a chave ja era utilizada em outro repositorio, a melhor pratica adotada foi gerar uma nova chave SSH dedicada e configurar um alias no arquivo de configuracao do SSH.

**Passo 1: Gerar nova chave**
```bash
ssh-keygen -t ed25519 -C "email@exemplo.com" -f ~/.ssh/id_ed25519_repos_sala
```

**Passo 2: Adicionar a chave publica ao GitHub**
O conteudo do arquivo `~/.ssh/id_ed25519_repos_sala.pub` foi adicionado nas configuracoes da conta do GitHub (SSH and GPG keys).

**Passo 3: Configurar o arquivo `~/.ssh/config`**
Foi adicionado um alias para forcar o uso da nova chave especificamente para este repositorio:
```text
Host github-repos-sala
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_repos_sala
    IdentitiesOnly yes
```

**Passo 4: Atualizar o endereco remoto**
A URL do repositorio local foi alterada para usar o alias criado:
```bash
git remote set-url origin git@github-repos-sala:wellington-cpu/Repos_de_Sala.git
```

---

## 3. Erro de non-fast-forward no Git Push

### Sintoma
Apos corrigir o problema de permissao, o comando `git push origin main` retornou a rejeicao:
`! [rejected] main -> main (non-fast-forward)`

### Causa
O historico do repositorio local e o historico do repositorio remoto divergiram. O GitHub possuia um commit novo (uma pagina de estatisticas) que a maquina local ainda nao tinha. O Git bloqueou o envio para evitar a perda do trabalho remoto.

### Solucao
O primeiro passo foi baixar as referencias remotas sem alterar os arquivos locais:
```bash
git fetch origin
```

Em seguida, foi verificado que os historicos continham commits diferentes:
```bash
git log --oneline HEAD..origin/main
git log --oneline origin/main..HEAD
```

Para integrar os commits remotos antes de enviar os locais, foi utilizado o comando `pull` com a flag `--rebase`, que reaplica os commits locais no topo do historico remoto, mantendo o historico linear:
```bash
git pull --rebase origin main
```

Como nao houve conflitos, o envio foi concluido com sucesso:
```bash
git push origin main
```

---

## Resumo das Boas Praticas Adotadas

1. Arquivos de configuracao, como o `.gitignore`, devem ser criados usando editores de texto ou redirecionadores (`>`, `>>`), nunca colados diretamente no prompt de comando.
2. Para o GitHub, e recomendavel utilizar chaves SSH pessoais ou chaves dedicadas com aliases no arquivo `config` do SSH, em vez de reutilizar Deploy Keys (que sao restritas e de uso unico).
3. Sempre executar `git pull` (preferencialmente com `--rebase`) antes de fazer um `git push` para garantir que o historico local esteja atualizado com o remoto e evitar erros de `non-fast-forward`.
