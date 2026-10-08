# 💾 Sistema de Arquivos com i-nodes

**Projeto acadêmico de Sistemas Operacionais**, desenvolvido em **Python**, que simula um sistema de arquivos com armazenamento persistente em um disco virtual de **128 MiB**.

A implementação utiliza **i-nodes, bitmaps de ocupação e blocos de 2.048 bytes** para organizar arquivos e diretórios. Conta com um terminal interativo, suporte a links simbólicos e caminhos absolutos e relativos.

### ✨ Funcionalidades

- Criar, ler, editar, copiar, mover e remover arquivos.
- Criar e navegar em diretórios.
- Gerenciar blocos de dados e i-nodes livres ou ocupados.
- Criar e utilizar links simbólicos.
- Persistir os dados entre execuções no arquivo `filesystem.img`.
- Executar **32 testes automatizados** cobrindo operações e situações de erro.


## Requisitos

- Python 3.10 ou superior.
- Não é necessário instalar bibliotecas adicionais.

## Execução

Baixe ou clone este repositório e abra o terminal na pasta que contém `main.py`.

Execute:

```bash
python main.py
```

No Windows, também pode ser utilizado:

```bash
py main.py
```

No Linux:

```bash
python3 main.py
```

Após iniciar, o programa apresenta o terminal:

```text
myfs>
```

Digite os comandos nesse terminal. Para encerrar, utilize:

```text
exit
```

O programa também encerra ao receber `Ctrl+C` ou o fim da entrada.

Na primeira execução, é criado o arquivo `filesystem.img`, que representa o disco virtual. Execute o programa na mesma pasta para continuar utilizando os dados armazenados.

Os caminhos informados nos comandos pertencem ao disco virtual, não às pastas do computador.

## Arquivos do projeto

| Arquivo | Descrição |
|---|---|
| `main.py` | Implementação do sistema e terminal de comandos |
| `README.md` | Instruções de execução e funcionamento |
| `testes.py` | Testes automatizados |
| `VALIDACAO.md` | Descrição dos testes e resultados |
| `roteiro_demonstracao.txt` | Sequência de comandos para demonstração |

## Organização do disco

| Estrutura | Configuração |
|---|---|
| Capacidade total | 134.217.728 bytes — 128 MiB |
| Tamanho dos blocos | 2.048 bytes |
| Quantidade de blocos | 65.536 |
| Área de gerenciamento | 2.048 blocos — 4 MiB |
| Área de dados | 63.488 blocos — 124 MiB |
| Quantidade máxima de i-nodes | 8.192 |
| Apontadores diretos por i-node | 12 |

Os primeiros 4 MiB armazenam os i-nodes e os mapas de ocupação. Os 124 MiB restantes são compartilhados por arquivos, diretórios e links.

Os mapas `bitmap_inode` e `bitmap_bloco` são listas nas quais:

- `0` indica uma posição livre;
- `1` indica uma posição ocupada.

O deslocamento de um bloco na imagem é calculado por:

```python
numero_do_bloco * TAM_BLOCO
```

Os metadados são serializados com `pickle`. Os primeiros oito bytes da imagem indicam o tamanho dessa serialização. Antes da gravação, o programa verifica se os metadados cabem na área reservada.

## Estrutura dos i-nodes

Cada i-node contém:

- Nome e tipo da entrada.
- Criador e dono.
- Tamanho do conteúdo em bytes.
- Datas de criação e modificação.
- Permissões de acesso.
- Doze apontadores para blocos.
- Apontador `proximo`, utilizado no encadeamento.

Os doze apontadores armazenam até 24 KiB de conteúdo. Quando esse espaço é insuficiente, outro i-node é reservado e ligado ao anterior pelo campo `proximo`.

O tamanho total do conteúdo fica no i-node principal. Apontadores não utilizados recebem o valor `-1`.

Criador e dono são definidos como `usuario`. O campo `permissoes` armazena `rwxr-x`, representando as permissões do dono e dos demais usuários. Nesta implementação, esses campos são apenas armazenados: não há autenticação nem verificação de permissões nos comandos.

## Comandos de arquivos

| Comando | Operação |
|---|---|
| `touch arquivo` | Cria um arquivo vazio ou atualiza a data de modificação de uma entrada existente |
| `rm arquivo` | Remove um arquivo ou link |
| `echo "texto" > arquivo` | Cria um arquivo ou substitui seu conteúdo |
| `echo "texto" >> arquivo` | Acrescenta conteúdo ou cria o arquivo se necessário |
| `cat arquivo` | Exibe o conteúdo |
| `cp origem destino` | Copia o conteúdo de um arquivo |
| `mv origem destino` | Move ou renomeia uma entrada |
| `ln -s alvo link` | Cria um link simbólico |

No comando `echo`, use aspas quando o texto contiver espaços. Os operadores `>` e `>>` podem ser utilizados com ou sem espaços ao redor.

O `echo` não acrescenta uma quebra de linha ao conteúdo armazenado. O `cat` acrescenta uma quebra de linha apenas na exibição.

O `cp` pode substituir o conteúdo de um arquivo existente, mas recusa origem e destino que correspondam ao mesmo i-node.

O `mv` recusa destinos já ocupados. Quando o destino é um diretório, a entrada é movida para dentro dele, mantendo seu nome.

## Comandos de diretórios

| Comando | Operação |
|---|---|
| `mkdir diretorio` | Cria um diretório |
| `rmdir diretorio` | Remove um diretório vazio |
| `ls diretorio` | Lista as entradas |
| `cd diretorio` | Altera o diretório atual |
| `mv origem destino` | Move ou renomeia um diretório |
| `ln -s alvo link` | Cria um link simbólico para um diretório |

Sem argumento, `ls` lista o diretório atual.

Cada diretório armazena uma associação entre nomes e números de i-node. Também contém:

- `.`: referência ao próprio diretório.
- `..`: referência ao diretório pai.

Na raiz, ambas apontam para o i-node 0.

O sistema impede remover a raiz, remover o diretório atual e alterar diretamente as entradas `.` e `..`. Também impede mover um diretório para dentro dele mesmo ou de seus descendentes.

## Caminhos e links

Os comandos aceitam caminhos absolutos e relativos:

```text
/documentos/arquivo.txt
arquivo.txt
./arquivo.txt
../arquivo.txt
```

Caminhos absolutos começam na raiz. Caminhos relativos começam no diretório atual. Os diretórios intermediários precisam existir.

Nomes com espaços devem ser colocados entre aspas:

```text
mkdir "meus documentos"
```

Um link simbólico armazena o caminho do alvo. Quando esse caminho é relativo, ele é interpretado a partir do diretório que contém o link.

Remover um link com `rm` não remove seu alvo. Na origem de `mv`, o link também é tratado como uma entrada própria.

Links podem apontar para arquivos ainda inexistentes. `touch` e `echo` podem criar esse alvo, desde que o diretório pai exista.

A resolução de um caminho permite até 40 expansões de links. Ao ultrapassar esse limite, o comando é recusado.

Por proteção, operações como `rm link/`, `rmdir link/` e `mv link/ destino` são recusadas quando a última entrada é um link. Para remover apenas o link, utilize `rm link`.

## Persistência e tratamento de erros

Os dados são armazenados em `filesystem.img` e recuperados na próxima execução.

Cada comando de alteração utiliza `transacao()`, que:

1. Guarda uma cópia dos i-nodes e mapas em memória.
2. Prepara os blocos modificados sem gravá-los imediatamente.
3. Restaura o estado anterior se ocorrer um erro durante a preparação.
4. Grava os blocos e metadados após a conclusão das verificações.

Antes de substituir conteúdo, `escrever()` verifica se existem blocos e i-nodes suficientes, considerando os recursos que o próprio arquivo pode reutilizar.

Os comandos `cat`, `ls` e `cd` não regravam os metadados. Cada comando de alteração concluído já deixa seus dados persistidos.

Ao iniciar novamente, o diretório atual é a raiz.

## Limites da implementação

- Nomes novos podem ter até 255 bytes em UTF-8.
- Muitos arquivos pequenos podem esgotar os i-nodes antes dos blocos.
- Não há cópia recursiva de diretórios.
- As permissões são armazenadas, mas não são aplicadas.
- O acréscimo com `echo >>` relê e prepara o conteúdo completo.
- A imagem deve ser utilizada por uma única instância por vez.
- Não há recuperação automática de queda de energia ou interrupção durante a gravação final.
- A imagem utiliza `pickle`; devem ser abertas apenas imagens de procedência conhecida.

## Exemplo de uso

```text
mkdir documentos
cd documentos
echo "conteudo inicial" > arquivo.txt
echo " e mais conteudo" >> arquivo.txt
cp arquivo.txt copia.txt
mv copia.txt renomeado.txt
ln -s arquivo.txt atalho.txt
cat atalho.txt
rm atalho.txt
cat arquivo.txt
cd ..
ls documentos
exit
```

Ao executar novamente, consulte o arquivo:

```text
cat /documentos/arquivo.txt
```

Resultado esperado:

```text
conteudo inicial e mais conteudo
```

## Testes automatizados

Na pasta do projeto, execute:

```bash
python -m unittest -v testes
```

A suíte contém 32 testes com subcasos. Os testes utilizam imagens temporárias e não alteram o disco da demonstração.

O teste de capacidade utiliza temporariamente cerca de 128 MiB em disco e algumas centenas de MiB de memória.

## Demonstração automática

O arquivo `roteiro_demonstracao.txt` contém uma sequência de comandos de arquivos, diretórios e links.

Execute o roteiro em uma pasta de demonstração nova, sem uma imagem `filesystem.img` existente, pois ele utiliza nomes fixos.

No Prompt de Comando do Windows, Linux ou macOS:

```bash
python main.py < roteiro_demonstracao.txt
```

No PowerShell:

```powershell
Get-Content roteiro_demonstracao.txt | python main.py
```

Depois, abra o programa novamente e execute:

```text
cat /destino/demonstracao/arquivo.txt
```

Resultado esperado:

```text
conteudo legal e mais conteudo
```

---

### 👤 Perfil

[Antonio Ollé Ramos](https://github.com/antonioolleramos) · Engenharia de Computação — FURG.
