# Testes do sistema de arquivos

## Ambiente

- Data da execução: 21/09/2026.
- Sistema operacional: Linux.
- Versão do Python: 3.12.
- Bibliotecas utilizadas: biblioteca padrão do Python.

A execução nativa no Windows não foi verificada nesta etapa.

## Resultado

Foram executados **32 testes automatizados, todos aprovados, sem falhas ou erros**.

A suíte também foi executada nos arquivos extraídos do ZIP de entrega.

Para repetir os testes:

```bash
python -m unittest -v testes
```

Cada teste utiliza uma imagem temporária. O arquivo `filesystem.img` utilizado pelo usuário não é modificado.

## Operações verificadas

| Grupo | Verificações |
|---|---|
| Arquivos | Criação, leitura, escrita, acréscimo, cópia, movimentação e remoção |
| Diretórios | Criação, listagem, navegação, movimentação e remoção |
| Caminhos | Absolutos, relativos, `.` e `..`, espaços, Unicode e barras repetidas |
| Links | Arquivos e diretórios, alvos relativos e absolutos, alvos ausentes, ciclos e movimentação |
| Metadados | Tamanho em bytes, datas, criador, dono e permissões armazenadas |
| Persistência | Reabertura da imagem e execução em processos diferentes |
| Capacidade | Esgotamento dos blocos e dos i-nodes |
| Tratamento de erros | Sintaxe inválida, destinos ocupados, tipos incorretos e falta de recursos |
| Terminal | Continuidade após erros, encerramento por EOF e `Ctrl+C` |
| Demonstração | Execução do roteiro incluído no pacote |

## Blocos e encadeamento

Foram testados conteúdos com os seguintes tamanhos:

```text
0, 1, 2.047, 2.048, 2.049, 24.576, 24.577 e 48.000 bytes
```

Esses casos verificam arquivos vazios, limites dos blocos de 2.048 bytes e a necessidade de encadear outro i-node após os primeiros 24.576 bytes.

Também foi verificada a liberação de blocos e i-nodes após sobrescrever ou remover arquivos.

## Disco cheio e esgotamento de i-nodes

O teste de capacidade gravou um arquivo de **130.021.376 bytes**, ocupando todos os blocos de dados disponíveis, exceto o bloco da raiz.

O conteúdo foi conferido antes e depois de reabrir a imagem. As tentativas de acrescentar dados ou criar outro arquivo com conteúdo foram recusadas, preservando os dados existentes.

Outro teste ocupou os **8.192 i-nodes**, incluindo os utilizados na continuação do diretório raiz. As operações que exigiam novos i-nodes foram recusadas sem perda do conteúdo anterior.

## Integridade e preservação dos dados

A verificação percorre os diretórios e confere:

- Referências `.` e `..`.
- Correspondência entre entradas e i-nodes.
- Tamanhos dos conteúdos.
- Encadeamento de i-nodes.
- Ocupação dos blocos.
- Ausência de blocos utilizados indevidamente por mais de uma entrada.
- Ausência de recursos alocados sem referência.

Foram simuladas falhas após preparar partes de uma criação ou movimentação e situações em que os metadados ultrapassariam a área reservada.

Em casos selecionados, o SHA-256 da imagem inteira foi comparado antes e depois da operação recusada. A comparação confirmou que os bytes persistidos permaneceram iguais.

## Sequência de operações

Foi executada uma sequência com **240 escolhas de operações**, utilizando uma semente fixa para permitir sua reprodução.

A sequência combinou escrita, acréscimo, cópia, movimentação e remoção. Os resultados foram comparados com um modelo de referência, com verificações de integridade e reaberturas periódicas da imagem.

## Persistência e terminal

Foram verificados:

- Recuperação do conteúdo após fechar e reabrir a imagem.
- Persistência entre duas execuções independentes.
- Recuperação de um arquivo que ocupa praticamente toda a área de dados.
- Carregamento de uma imagem com tabela de 1.024 i-nodes e ampliação para 8.192.
- Continuidade do terminal após comandos inválidos.
- Encerramento por fim da entrada e `Ctrl+C`.
- Importação do módulo sem iniciar o terminal ou criar uma imagem.
- Execução do roteiro de demonstração.

## Limites da verificação

Os resultados correspondem aos cenários descritos neste documento.

As permissões foram verificadas como campos armazenados. Não há autenticação ou controle de acesso na implementação.

Não foram realizados testes de queda real de energia, acesso simultâneo à mesma imagem ou execução nativa no Windows. O mecanismo de transação protege a preparação dos comandos, mas não oferece recuperação automática de interrupções durante a gravação final.
