# Texto em português (pt-BR)

O ASD-STE100 é um padrão para inglês. Não existe versão oficial em português. Este arquivo adapta as regras estruturais da skill para pt-BR. As regras lexicais dependem do dicionário do ASD, que também é só em inglês. Em português, elas valem apenas como direção.

Leia este arquivo antes da primeira reescrita em pt-BR de uma sessão.

## Regra de ouro

Texto em português sai em português. Nunca traduza para inglês para aplicar o STE e depois traduza de volta. A tradução muda palavras, e cada palavra trocada pode mudar o sentido.

## Esta skill ou a unimed-vr-escrita-tecnica

| Situação | Skill |
|---|---|
| Um agente, uma ferramenta ou um sistema lê o texto sem poder perguntar nada: descrição de ferramenta, mensagem de erro, prompt, instrução entre agentes, status | `asd-ste100` |
| Uma pessoa lê o texto: ADR, spec, README, descrição de MR, comentário no Jira, e o problema é LLM-ês, jargão vazio ou falta de template | `unimed-vr-escrita-tecnica` |

As duas combinam. Um prompt em português pode passar pela lista negra da `unimed-vr-escrita-tecnica` e depois pelas regras abaixo.

## Regras estruturais adaptadas

A coluna "Linter" mostra a ID da regra no `scripts/ste-lint.py`. As IDs são as mesmas do inglês, então `--disable` vale para os dois idiomas.

| Regra | Faça | Não faça | Linter |
|---|---|---|---|
| Voz ativa | "O agente apaga o arquivo." | "O arquivo é apagado." "Apaga-se o arquivo." | `passive-voice` (aviso) |
| Sem locução verbal inflada (equivale ao phrasal verb) | "Use o cache." "Inicie o job." "Contate o time." | "Faça uso do cache." "Dê início ao job." "Entre em contato com o time." | `phrasal-verb` |
| Sem verbo-suporte (equivale à nominalização) | "Valide o arquivo." | "Realize a validação do arquivo." "Proceda à validação." | `nominalization` |
| Sem gerundismo | "Vamos enviar o relatório amanhã." | "Vamos estar enviando o relatório amanhã." | `gerundism` |
| Tempo simples | "O job falhou." | "O job tem falhado." (veja a exceção abaixo) | `present-perfect` (aviso) |
| Sem ponto e vírgula | Duas frases | Qualquer `;` | `semicolon` |
| Tamanho da frase | ≤ 20 palavras em instrução, ≤ 25 em descrição | Frase com várias orações encaixadas | `long-sentence` |
| Uma instrução por frase | "Abra o arquivo. Leia a linha 3." | "Abra o arquivo e leia a linha 3, depois confira se bate." | não verifica |
| Sem adjetivo de venda | O número ou o comportamento | robusto, poderoso, de ponta, fluido, simplesmente | `marketing-adjective` |
| Lista sem item pendurado | "- Configure o alvo." | "- Configure o alvo e" | `dangling-conjunction` |
| Manter a modalidade | "A requisição pode ter falhado." continua assim | "A requisição falhou." | nunca aponta |

## Diferenças que mudam o sentido em português

1. **Pretérito perfeito composto não é o present perfect.** "O job tem falhado" diz que o job falha de forma repetida até agora. "O job falhou" diz que ele falhou uma vez. Quando a repetição é o fato, mantenha o tempo composto e registre em `Mantido como está:`.
2. **Sujeito oculto.** O português permite omitir o sujeito. Omita só quando o sujeito é o mesmo da frase anterior. Quando o sujeito muda, escreva o sujeito. "O agente lê o log. Se encontrar erro, abre um ticket." está certo. "O agente envia o lote ao servidor. Se recusar, tenta de novo." é ambíguo: quem recusa?
3. **"o mesmo" como pronome.** "Caso seja detectado um conflito, o mesmo será resolvido" obriga o leitor a procurar o antecedente. Repita o substantivo: "o conflito".
4. **Passiva com "-se".** "Recomenda-se reiniciar o serviço" esconde quem recomenda e quem reinicia. Escreva "Reinicie o serviço." O imperativo reflexivo ("Certifique-se de que…") é instrução, não passiva.
5. **"pode estar + gerúndio" é ressalva.** "O serviço pode estar falhando" afirma uma suspeita. Não é gerundismo e o linter não aponta. "Vou estar enviando" é gerundismo.
6. **Cadeia de "de".** Em inglês, o grupo nominal empilha substantivos: "agent task queue priority handler". Em português, ele vira uma cadeia de "de": "o handler da prioridade da fila de tarefas do agente". Use no máximo três substantivos ligados por "de". O linter não verifica.
7. **Uma forma de instrução por documento.** Escolha o imperativo ("Execute o script") ou o infinitivo ("Executar o script"). Use a mesma forma em todo o documento. Não misture com "você deve executar".

## Um verbo por ação (direção, não regra)

O linter aponta a troca de sinônimos dentro de um mesmo arquivo nestes grupos, já conjugados ("verifique", "removido", "corrija"):

- verificar, conferir, checar, validar, confirmar
- excluir, remover, apagar, deletar
- iniciar, começar
- mostrar, exibir
- usar, utilizar, empregar
- corrigir, consertar
- enviar, mandar, transmitir
- alterar, modificar, mudar

Escolha um verbo por ação e use sempre o mesmo.

## O que o linter pt-BR não pega

- Cadeia de "de", sujeito oculto ambíguo, "o mesmo" como pronome e mais de uma instrução por frase. Verifique essas regras na leitura.
- Verbos irregulares fora das listas e locuções que não estão na regra `phrasal-verb`.
- Ressalva empilhada ("é importante ressaltar que talvez possa"). Por decisão de projeto, o linter nunca aponta modalidade.

Há dois falsos positivos conhecidos, ambos em regras de aviso. O primeiro é o adjetivo em -ado ou -ido depois de "ser", como em "o repositório é privado". O segundo é o verbo reflexivo terminado em -a, como "lembra-se".

O idioma é detectado por arquivo, pela contagem de palavras funcionais ("não", "que", "para" contra "the", "and", "of"). Em empate, o linter usa inglês. Para forçar o perfil, use `--lang pt` ou `--lang en`.
