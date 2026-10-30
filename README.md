<div align="center">

# Utilidades — Seaka-bot

**Comandos de utilidade do Sea pro BDFD:** prefixo por servidor, say, votação com botões, calculadora, contador de letras, timestamp unix, pesquisa no Google e mais. Cada arquivo tem `$c[]` explicando o que cada parte faz.

[Outros projetos do Seaka](https://github.com/seaka-bot)

</div>

> [!NOTE]
> **Código de acervo.** Saiu do Sea, um bot que rodou no BDFD. Os comandos estão como eram, com quatro mudanças: os emojis viraram dinâmicos (veja [a aula 2](#aula-2-emojis-dinâmicos)), o `==calc` avisa como usar quando vem vazio, o `==dizer` perdeu os `$async`/`$await` (não existem no BDFD) e o `==unix` foi reescrito porque a conta antiga errava a data.

## Comandos

| Arquivo | Gatilho no BDFD | O que faz |
|---|---|---|
| `prefixo/prefixo.bdscript2` | `s` | `s*prefixo !` troca o prefixo do servidor. Depois disso o bot responde a `s!`. |
| `prefixo/say.bdscript2` | `s` | `s*say texto` faz o bot mandar o texto e apaga o seu comando. Só admin. |
| `prefixo/poll.bdscript2` | `s` | `s*poll 3` começa uma votação com 3 opções (até 10). Só admin. |
| `oninteraction/poll.bdscript2` | `$onInteraction` | Os botões da votação: configurações, formulário, voto e barras de resultado. |
| `prefixo/meia-noite.bdscript2` | `s*meia noite` | Quanto falta pra meia-noite no horário de Brasília. Só o dono do bot. |
| `igual/calc.bdscript2` | `==calc` | `==calc 2+2*3` faz a conta. Só o dono do bot. |
| `igual/contar.bdscript2` | `==contar` | `==contar texto` diz quantos caracteres o texto tem. |
| `igual/emojis.bdscript2` | `==emoji` | Lista os emojis do servidor. |
| `igual/dizer.bdscript2` | `==dizer` | `==dizer @pessoa texto` manda o texto com o nome e a foto da pessoa, por webhook. Só admin. |
| `igual/pesquisa.bdscript2` | `==pesquisa` | `==pesquisa texto` devolve o link da busca no Google. |
| `igual/ping.bdscript2` | `==ping` | Latência e quando a hospedagem do BDFD expira. |
| `igual/unix.bdscript2` | `==unix` | `==unix 25 12 2026` mostra o timestamp da data, meia-noite em Brasília. |

Todos são **BDScript 2**. Os três da pasta `prefixo/` que têm gatilho `s` dividem o mesmo gatilho: cada um confere sozinho se é com ele (veja [a aula 1](#aula-1-vários-comandos-no-mesmo-gatilho)).

## Instalação

1. **Crie as variáveis** no BDFD:

   | Variável | Padrão | Pra que serve |
   |---|---|---|
   | `Prefixo` | `*` | O prefixo de cada servidor. Com `*`, os comandos ficam `s*say`, `s*poll`… |
   | `Emojis` | *(o ID do servidor do emoji)* | Onde o bot procura os emojis. Mais de um servidor: IDs separados por `-`. |
   | `PoolVotes` | *(vazio)* | Os votos de cada votação. |

2. **Suba o emoji** `emojis/errado.png` num servidor onde o bot está, com o nome **`errado`**, e coloque o ID desse servidor na variável `Emojis`.
3. **Crie um comando por arquivo**, com o gatilho da tabela, e cole o código.

## Aula 1: vários comandos no mesmo gatilho

Jeito mais direto: cada comando com o seu gatilho, `s*say`, `s*poll`, `s*prefixo`. Funciona até um servidor trocar o prefixo pra `!`. Aí `s!say` não bate com gatilho nenhum, e você teria que criar cada comando de novo pra cada prefixo possível.

O Sea resolve com **um gatilho só, a letra `s`**, e cada comando confere o resto. É o começo do `poll`:

```
$var[Prefixo;$cropText[$message;$charCount[$getServerVar[Prefixo]];]]
$onlyIf[$var[Prefixo]==$getServerVar[Prefixo]; ]
$var[cmd;$replaceText[$message[1];$var[Prefixo];;1]]
$textSplit[poll pool votações votes; ] $var[i;1] $var[code;$repeatMessage[$getTextSplitLength;$|if[$|var[cmd\]==$|splitText[$|var[i\]\]\] $|var[triggerCMD\;true\] $|stop $|endif $|var[i%|{-SEMICOL-}%$|calculate[$|var[i\]+1\]\]]] $eval[$replaceText[$var[code];|;;-1]] $if[$var[triggerCMD]==] $stop $endif
```

Peça por peça, com a mensagem `s*poll 3`:

1. Com gatilho `s`, o `$message` é tudo depois do `s`: `*poll 3`.
2. `$getServerVar[Prefixo]` é o prefixo deste servidor, `*`. O `$charCount` diz quantos caracteres ele tem (1), e o `$cropText` corta esse tanto do começo da mensagem. `$var[Prefixo]` fica `*`.
3. O `$onlyIf` compara os dois. Se o servidor usa `!` e a pessoa digitou `s*`, o comando para aqui, em silêncio.
4. `$message[1]` é a primeira palavra, `*poll`. Tirando o prefixo, sobra o nome do comando: `$var[cmd]` fica `poll`.
5. O `$textSplit` separa os nomes que chamam este comando: `poll`, `pool`, `votações` e `votes`.
6. O `$repeatMessage` escreve um `$if` por nome, um embaixo do outro. As funções com `$|` e os `\]` não rodam ainda: são texto.
7. O `$replaceText[...;|;;-1]` tira todas as `|`, e o `$eval` roda o texto. Agora cada `$if` compara `cmd` com um nome. Se bater, `triggerCMD` vira `true` e o `$stop` encerra só o `$eval`.
8. No fim, se `triggerCMD` continua vazio, o nome não estava na lista e o comando para.

**Lembre:** todo comando com gatilho `s` roda em **toda** mensagem que começa com `s`. Por isso a checagem vem primeiro, antes de qualquer coisa pesada.

**Cuidado:** o `prefixo` não aceita `=`, porque os comandos da pasta `igual/` começam com `==`.

## Aula 2: emojis dinâmicos

Jeito mais direto: colar o emoji pronto, `<:errado:1056984769298190416>`. O número é o ID, e ele só existe naquele servidor. Se você trocar o emoji de servidor, ou se outra pessoa usar o seu código, o ID muda e o emoji some. Aí é caçar o ID em cada comando que usa ele.

Os comandos daqui procuram o emoji **pelo nome** nos servidores da variável `Emojis`. Este é o bloco do `say`, que usa um emoji só, o `errado`:

```
$var[emjSplit;$joinSplitText[⁂]]$textSplit[$getVar[Emojis];-]$var[emjLista;]$var[emjI;1]$var[emjCode;$repeatMessage[$getTextSplitLength;$|var[emjLista\;$|var[emjLista\] $|serverEmojis[$|splitText[$|var[emjI\]\]\; \]\]$|var[emjI\;$|calculate[$|var[emjI\]+1\]\]]]$eval[$replaceText[$var[emjCode];|;;-1]]
$textSplit[errado; ]$var[emjI;1]$var[emjCode;$repeatMessage[$getTextSplitLength;$|var[emjN\;$|splitText[$|var[emjI\]\]\]$|textSplit[$|var[emjLista\]\;<:$|var[emjN\]:\]$|textSplit[$|splitText[2\]\;>\]$|var[e_$|var[emjN\]\;<:$|var[emjN\]:$|splitText[1\]>\]$|textSplit[errado\; \]$|var[emjI\;$|calculate[$|var[emjI\]+1\]\]]]$eval[$replaceText[$var[emjCode];|;;-1]]$textSplit[$var[emjSplit];⁂]
```

Peça por peça:

1. `emjSplit` guarda o `$textSplit` que o comando já estava usando, com cada parte separada por `⁂`. O bloco usa `$textSplit` também, então no fim devolve o que estava lá.
2. **Primeira linha.** Separa os IDs da variável `Emojis` (`-`) e, com o mesmo truque do `$eval` da aula 1, junta o `$serverEmojis` de cada servidor em `emjLista`. Fica algo como `<:errado:1056984769298190416> <:correto:1056984728948973630> …`.
3. **Segunda linha.** `errado` é a lista de nomes que este comando usa. Pra cada nome, `emjN` é o nome da vez.
4. `$textSplit[$var[emjLista];<:errado:]` corta a lista bem onde o emoji começa. A parte 2 começa com o ID: `1056984769298190416> <:correto:…`.
5. `$textSplit[...;>]` corta no primeiro `>`. A parte 1 é só o ID.
6. `$var[e_errado;<:errado:ID>]` guarda o emoji pronto. Daqui pra frente o comando usa `$var[e_errado]`.
7. A última função, `$textSplit[$var[emjSplit];⁂]`, devolve o split do passo 1.

**Lembre:** os nomes precisam ser únicos nos servidores da variável. Dois emojis chamados `errado` e o bot pega o primeiro que achar.

**Cuidado:** o bloco roda a cada uso do comando. Por isso a lista de nomes tem só os emojis que aquele comando usa.

## Aula 3: data pra timestamp sem função pronta

O BDFD não tem uma função que transforme `25 12 2026` em timestamp. O `==unix` faz a conta na mão, em quatro passos, e cada um tem um `$c[]` no arquivo:

1. **Anos inteiros.** 365 dias por ano desde 1970, mais um por ano bissexto que já passou.
2. **Meses.** A lista `0 31 59 90 …` diz quantos dias o ano já andou no começo de cada mês.
3. **Fevereiro.** Ano bissexto e data depois de fevereiro: mais um dia.
4. **Segundos.** Dias × 86400, mais 3 horas, porque meia-noite em Brasília é 3 da manhã no horário universal.

**Lembre:** bissexto é de 4 em 4 anos, menos os de 100 em 100, mas os de 400 em 400 voltam a ser. Por isso 2000 foi bissexto e 2100 não vai ser.
