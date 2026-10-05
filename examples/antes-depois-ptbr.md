# Antes e depois em pt-BR

Exemplos originais, criados para esta adaptação. Eles seguem as regras de `references/ptbr.md`. A contagem de palavras separa por espaço (`text.split()`).

## Exemplo A: mensagem de erro

**Antes:**
> Pode ter ocorrido um erro durante o processamento da sua solicitação devido a uma possível incompatibilidade no formato de dados esperado, o que poderia ser causado por uma versão desatualizada do cliente.

**Violações:**
- Uma frase com três afirmações: um erro, uma incompatibilidade de formato e a versão do cliente.
- 32 palavras, acima do limite de 25 para descrição.
- Passiva com o agente escondido: "poderia ser causado".

Fora da lista: "pode ter ocorrido" e "poderia". São ressalvas. O sistema não sabe o que aconteceu, e a mensagem precisa continuar dizendo isso.

**Depois:**
> Sua solicitação pode ter falhado. A causa pode ser um formato de dados diferente do que o servidor espera. Uma versão desatualizada do cliente pode causar essa diferença. Verifique a versão do cliente.

## Exemplo B: descrição de ferramenta

**Antes:**
> Esta ferramenta robusta realiza a sincronização do estado entre os diversos backends que tenham sido configurados; caso seja detectado um conflito, o mesmo poderá ser resolvido automaticamente de acordo com a estratégia definida, ou então será encaminhado para revisão manual.

**Violações:**
- Adjetivo de venda: "robusta".
- Verbo-suporte: "realiza a sincronização".
- Ponto e vírgula.
- Tempo composto: "tenham sido configurados".
- Três passivas: "seja detectado", "ser resolvido", "será encaminhado".
- "o mesmo" como pronome.
- 40 palavras em uma frase.

**Depois:**
> A ferramenta sincroniza o estado entre os backends configurados. Se encontrar um conflito, a ferramenta lê a estratégia configurada. Se a estratégia permitir resolução automática, a ferramenta pode resolver o conflito sem um usuário. Se a ferramenta não resolver o conflito, ela envia o conflito para revisão manual.

"poderá ser resolvido" virou "pode resolver". A ressalva continua: a ferramenta não promete resolver.

## Exemplo C: instrução entre agentes

**Antes:**
> Assim que o job upstream tiver sido concluído e, desde que nenhum erro tenha sido gerado, o agente downstream deverá estar consumindo o artefato de saída, sendo importante ressaltar que artefatos parciais às vezes são produzidos em situações de timeout.

**Violações:**
- Tempo composto: "tiver sido concluído", "tenha sido gerado".
- Passiva com o agente escondido: "são produzidos".
- Três fatos em uma frase: a condição, a próxima ação e o aviso.
- 40 palavras, acima do limite de 20 para instrução.
- "deverá estar consumindo" é gerundismo com "dever". O linter não aponta essa forma, porque "deve estar + gerúndio" também expressa suspeita ("deve estar falhando"). Corrija na leitura.

**Depois:**
> Espere o job upstream terminar sem erros. Depois leia o artefato de saída. Atenção: um timeout às vezes produz um artefato parcial. Verifique se o artefato está completo antes de usá-lo.

"às vezes" ficou. É uma informação de frequência, não enfeite.

A última frase é **nova**. O original avisava sobre artefatos parciais, mas não dizia o que fazer. A verificação torna o aviso útil. Como é conteúdo acrescentado, ela aparece aqui em destaque. Se o silêncio do original foi intencional, corte a frase.
