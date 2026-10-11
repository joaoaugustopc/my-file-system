# MyFS

Trabalho prático de Sistemas Operacionais (DCC062) da UFJF.

A disciplina forneceu um simulador de um S.O. hipotético, com APIs prontas para disco, i-nodes e um VFS genérico. O trabalho era implementar um sistema de arquivos próprio em cima disso. Todo o código está em `myfs.c`. Os outros arquivos (`disk`, `inode`, `vfs`, `util` e `main`) são do simulador.

## Compilando

```
gcc -o simulador *.c
./simulador
```

Para testar no simulador:

1. `D` → `B` cria um arquivo de disco (ex.: `32cyl.dsk` com 32 cilindros) e `C` conecta esse disco.
2. `F` → `F` formata o disco. Informe o ID do MyFS (aparece em `F` → `L`) e o tamanho do bloco em setores. Depois, `M` monta o disco.
3. `I` abre o menu de arquivos, com as opções de abrir, escrever, ler e fechar.
4. Antes de sair, use `F` → `U` para desmontar.

## Como o disco fica organizado

- **Setor 0**: superbloco. Ele começa com a assinatura `MYFS`, seguida da struct `SuperBlock` (número de i-nodes, tamanho do bloco, início da área de dados, último cluster e o primeiro cluster livre). O mount usa essa assinatura para saber se o disco foi formatado com o MyFS.
- **Área de i-nodes**: começa em `inodeAreaBeginSector()`. Há um i-node para cada 8 blocos do disco, com no mínimo 8 e no máximo 1024. O i-node 1 é o diretório raiz.
- **Área de dados**: vem logo depois dos i-nodes e fica alinhada ao tamanho do bloco.

O espaço livre é controlado por uma lista encadeada dentro do próprio disco. O primeiro setor de cada cluster livre guarda o endereço do próximo cluster livre, e o superbloco guarda o início da lista. Para alocar um cluster, o programa tira o primeiro da lista, atualiza o superbloco e zera o setor.

O diretório raiz é um arquivo comum cujo conteúdo é uma sequência de `DirEntry` (número do i-node + nome com até 255 caracteres). Uma entrada pode ficar dividida entre dois blocos, e a leitura e a escrita tratam esse caso.

## Arquivos abertos

A tabela de arquivos abertos fica em memória e tem `MAX_FDS` (128) posições. Cada posição guarda o i-node, o cursor e o disco do arquivo. O descritor devolvido ao usuário é o índice da tabela + 1.

- `myFSOpen` procura o nome no diretório raiz. Se não encontra, pega um i-node livre, cria o arquivo e adiciona a entrada no diretório.
- `myFSRead` e `myFSWrite` andam bloco a bloco a partir do cursor e depois avançam o cursor. A escrita aloca novos clusters quando o arquivo cresce e atualiza o tamanho no i-node.
- `myFSClose` libera a posição na tabela.

## Grupo

- Caio Louback
- Diogo Carminatti
- João Augusto Pilato
- Karen Jardim
