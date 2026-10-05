# Skill ASD-STE100: Simplified Technical English para saída de agentes

Skill do Claude Code que reescreve texto denso ou ambíguo no padrão [ASD-STE100 Simplified Technical English](https://www.asd-ste100.org/) (STE), em inglês ou em português do Brasil. O STE é a linguagem controlada que a indústria aeroespacial e de defesa criou para que ninguém leia errado uma instrução de manutenção de aeronave.

A skill usa a mesma disciplina para outro leitor: um **agente de IA** que lê a saída de outro agente, a descrição de uma ferramenta, uma mensagem de erro ou uma instrução entre agentes. Nesses casos, não há uma pessoa para resolver a ambiguidade.

> **Fork com suporte a pt-BR.** Este fork acrescenta ao original:
>
> - um perfil de português do Brasil no linter (`--lang auto|en|pt`) e a regra extra `gerundism`;
> - a referência [`references/ptbr.md`](references/ptbr.md), com as regras adaptadas e os casos em que o português muda o sentido;
> - os exemplos [`examples/antes-depois-ptbr.md`](examples/antes-depois-ptbr.md).
>
> A saída do linter para texto em inglês é idêntica à do original: [danyuchn/asd-ste100-skill](https://github.com/danyuchn/asd-ste100-skill).

## Por que STE, e por que para agentes

O STE existe porque uma instrução mal lida numa aeronave pode matar. Os leitores eram, muitas vezes, técnicos que não tinham o inglês como língua materna e não podiam ligar para o autor. A solução do padrão: uma palavra com um só sentido, voz ativa, tempos simples, uma instrução por frase, frases curtas e nenhuma palavra omitida.

Um agente que lê a saída de outro agente está numa situação parecida. Ele não tem como perguntar "você quis dizer X ou Y?". As regras que impedem um mecânico de ler errado um torque de aperto também impedem um agente de ler errado a descrição de uma ferramenta.

## Antes e depois

| Antes | Depois |
|---|---|
| "Pode ter ocorrido um erro durante o processamento da sua solicitação devido a uma possível incompatibilidade no formato de dados esperado, o que poderia ser causado por uma versão desatualizada do cliente." | "Sua solicitação pode ter falhado. A causa pode ser um formato de dados diferente do que o servidor espera. Uma versão desatualizada do cliente pode causar essa diferença. Verifique a versão do cliente." |
| "This tool will attempt to synchronize state across the various backends that have been configured, and if a conflict is detected it may resolve it automatically depending on the strategy that has been set, or otherwise it will surface the conflict for manual review." | "The tool tries to synchronize state across the configured backends. If it finds a conflict, it reads the configured strategy. If the strategy allows automatic resolution, the tool may resolve the conflict without a user. If the tool does not resolve the conflict, it reports the conflict for manual review." |

Mais exemplos em [`examples/antes-depois-ptbr.md`](examples/antes-depois-ptbr.md) (pt-BR) e [`examples/before-after.md`](examples/before-after.md) (inglês, com ilustrações das regras oficiais do STE).

## O que a skill faz

1. Escolhe um modo. O **estrito** vale para procedimentos, mensagens de erro e descrições de ferramenta. O **com sabor de STE** vale para READMEs, descrições de PR e textos explicativos. Ele mantém a disciplina das frases, mas não trava o vocabulário.
2. Lê o texto uma vez para entender o sentido.
3. Aponta cada violação, frase por frase. Exemplos: palavra ambígua, tempo composto, voz passiva sem agente claro, várias instruções numa frase, grupo nominal longo, palavra omitida, frase longa demais, locução verbal, verbo-suporte, ponto e vírgula, ressalva empilhada e adjetivo de venda.
4. Reescreve cada frase apontada sem perder nenhum fato, condição ou limite de escopo. Se a versão curta perder precisão, a skill mantém a versão longa e avisa.
5. Devolve só o texto reescrito: sem introdução, sem anunciar o modo, sem resumo das mudanças. Quando mantém algo de propósito, acrescenta uma linha `Mantido como está:` (`Kept as-is:` em inglês).

Para ver o raciocínio, peça "mostra o diff" ou "quais regras foram violadas". A skill devolve uma tabela de antes e depois com o nome de cada regra.

Texto em português sai em português. A skill nunca traduz para o inglês e de volta, porque cada palavra traduzida pode mudar o sentido.

## Linter

O `scripts/ste-lint.py` é um linter determinístico, só com a biblioteca padrão do Python. Ele verifica as regras estruturais, que são mecânicas: dá para apontar a palavra ou o sinal que quebra cada uma. As regras que dependem do dicionário do ASD ficam como recomendação. As que pedem bom senso ficam com você.

```bash
python scripts/ste-lint.py ARQUIVO.md             # idioma detectado por arquivo
python scripts/ste-lint.py --lang pt ARQUIVO.md   # força o perfil pt-BR
echo "texto" | python scripts/ste-lint.py --json  # saída estruturada
python scripts/ste-lint.py --baseline 5 ARQUIVO   # tolera 5 violações graves
python scripts/ste-lint.py --disable passive-voice,present-perfect ARQUIVO
python scripts/ste-lint.py --selftest
```

| Regra | Inglês | Português | Nível |
|---|---|---|---|
| `semicolon` | `;` | `;` | grave |
| `long-sentence` | mais de 25 palavras | mais de 25 palavras | grave |
| `phrasal-verb` | spin up, reach out, dive into, kick off… | fazer uso de, dar início a, entrar em contato, a fim de… | grave |
| `nominalization` | perform an analysis of | realizar a validação, proceder à análise | grave |
| `marketing-adjective` | seamless, robust, cutting-edge… | robusto, poderoso, de ponta, fluido… | grave |
| `gerundism` | não se aplica | vou estar enviando, estaremos analisando | grave |
| `synonym-rotation` | check/verify/confirm… | verificar/conferir/validar…, já conjugados | grave |
| `dangling-conjunction` | item de lista que termina em and/or | item de lista que termina em e/ou | grave |
| `passive-voice` | is removed | foi removido, recomenda-se | aviso |
| `present-perfect` | has run | tem falhado, tenha sido gerado | aviso |

Violações graves fazem o linter sair com código 1 quando passam do `--baseline` (padrão 0). Avisos nunca reprovam a execução. As IDs das regras são as mesmas nos dois idiomas, então `--disable` vale para os dois.

O linter **nunca** aponta ressalvas nem modalidade ("may have failed", "pode ter falhado", "pode estar falhando", "talvez"). A confiança do autor faz parte do conteúdo. Uma reescrita que transforma uma suspeita em fato muda o que o texto afirma.

### Limites do linter

- Ele verifica padrões estruturais. Não compara o original com a reescrita e não prova que o sentido ficou igual. Zero violações quer dizer apenas que as checagens configuradas não acharam problema.
- Não verifica grupo nominal longo (precisa de análise gramatical). Em português, também não verifica cadeia de "de", sujeito oculto ambíguo nem "o mesmo" como pronome.
- Só conhece as locuções da sua lista. "take off the panel" passa sem aviso.
- O idioma é detectado pela contagem de palavras funcionais ("não", "que", "para" contra "the", "and", "of"). Em empate, o linter usa inglês.
- A regra `dangling-conjunction` lê listas Markdown com marcador `-`, `*`, `+`, `1.` ou `1)`, com zero a três espaços antes. Ela não lê listas dentro de citação nem toda a semântica de listas aninhadas. O arquivo [`examples/linter-edge-cases.md`](examples/linter-edge-cases.md) é um caso de teste inválido de propósito. `python scripts/ste-lint.py examples/linter-edge-cases.md` deve apontar duas violações.

A própria documentação da skill não passa limpa no linter: as tabelas citam os padrões que proíbem, e algumas frases são longas. Use `python scripts/ste-lint.py --baseline 41 SKILL.md` e leia o resultado como exemplo, não como defeito.

## O dicionário oficial fica de fora

A skill **não** reproduz o dicionário oficial do ASD, com cerca de 900 palavras aprovadas. O padrão é gratuito, mas não pode ser redistribuído. A Issue 9 só permite reprodução com autorização escrita do ASD ou por oito categorias de organização, e este projeto não está em nenhuma delas. A skill aplica o *princípio* do dicionário: a palavra mais simples disponível, usada sempre do mesmo jeito. Para documentação com conformidade STE certificada, use o padrão oficial.

O STE é um padrão para inglês e não tem versão oficial em português. O perfil pt-BR adapta as regras estruturais. As regras lexicais valem só como direção.

Resumo completo das regras e fontes: [`references/writing-rules.md`](references/writing-rules.md) (inglês) e [`references/ptbr.md`](references/ptbr.md) (pt-BR).

## Instalação

### Pelo CLI skills

```bash
npx skills add gabriel-sousa99/asd-ste100-skill -g -a claude-code
```

O comando instala a skill para o Claude Code em todos os projetos. Para atualizar, rode `npx skills update`. O CLI envia telemetria anônima de instalação (nome da skill e horário). Para desligar, use `DISABLE_TELEMETRY=1`.

O `npx skills update` apaga e recria a pasta da skill. Se você editar a skill localmente, faça commit e push antes de atualizar.

### Por clone

```bash
git clone https://github.com/gabriel-sousa99/asd-ste100-skill ~/.claude/skills/asd-ste100
```

O clone deixa a skill disponível em todo projeto do Claude Code e atualiza com `git pull`. É a melhor opção para quem vai editar a skill.

## Uso

Peça para simplificar ou desambiguar um texto:

```
Aplica STE100 nesta mensagem de erro: …
Reescreve esta descrição de ferramenta para um agente não interpretar errado
Disambiguate this tool description
```

Você recebe só o texto reescrito. Para ver as regras aplicadas, acrescente "mostra o diff" ou "explica as mudanças".

## Escopo

Feita para: mensagens entre agentes, descrições de ferramentas e funções, mensagens de erro, prompts de sistema e qualquer texto que uma máquina ou um leitor não nativo precisa ler sem poder perguntar.

Não é para: texto criativo, texto de marketing ou qualquer texto em que a voz e a nuance são o objetivo. O STE é plano e literal de propósito.

Para eliminar LLM-ês de documentos que pessoas leem (ADR, spec, descrição de MR, Jira), use uma skill de escrita técnica.

Um limite vale desde já: a skill corrige a forma do texto, não o conteúdo. Um parágrafo sem nada a dizer sai curto, limpo e ainda vazio.

## Fontes

- [Site oficial do ASD-STE100](https://www.asd-ste100.org/)
- [ASD-STE100: sobre o STE](https://www.asd-ste100.org/about_STE.html)
- [ASD Europe: Simplified Technical English](https://www.asd-europe.org/standards-specifications/simplified-technical-english/)
- [Simplified Technical English na Wikipédia](https://en.wikipedia.org/wiki/Simplified_Technical_English)
- [TechScribe: ASD-STE100 Simplified Technical English](https://www.techscribe.co.uk/techw/asd-simplified-technical-english.htm)

## Licença

MIT. Veja [LICENSE](LICENSE). Projeto original de Dustin Yuchen Teng ([danyuchn/asd-ste100-skill](https://github.com/danyuchn/asd-ste100-skill)).
