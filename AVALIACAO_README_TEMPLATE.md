# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

**Grupo:** ______

| Integrante | RM | Turma |
|Lucas Franco de Godoy Fortes|561723|CCPO|
|Rafael Silva|565415|CCPO|
|Pedro Noronha|564572|CCPO|
| | | |
| | | |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 3 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | **26 testes, 0 falhas** |

---

## Parte 1 — Bugs encontrados

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | O agendamento duplicado não era identificado corretamente. | `AgendaService`, comparação de valores usando referência em vez de igualdade. | Alteração das comparações para `.equals()`. | `==` vs `.equals()` |
| bug02 | A mesma data/hora em objetos diferentes podia passar pela validação de conflito. | `AgendaService`, comparação incorreta de `LocalDateTime`. | Uso de `.equals()` para comparar os valores de data e hora. | Objetos e igualdade |
| bug03 | Era possível tentar agendar atendimento com data/hora passada. | `AgendaService.agendar()`, ausência da validação temporal. | Adicionada validação com `isBefore(LocalDateTime.now())`. | Regra de negócio / validação |
| bug04 | Atendimento concluído podia sofrer nova conclusão. | `Atendimento.concluir()`, ausência de validação de status. | Validação para permitir conclusão somente quando o status é `AGENDADO`. | Encapsulamento / regra de estado |
| bug05 | Atendimento cancelado podia ser concluído. | `Atendimento.concluir()`, transição de estado não validada. | Lançamento de `StatusInvalidoException` quando o status não é `AGENDADO`. | Máquina de estados |
| bug06 | Atendimento concluído podia ser cancelado. | `Atendimento.cancelar()`, ausência da validação de status. | Cancelamento permitido somente para atendimento `AGENDADO`. | Regra de negócio |
| bug07 | Atendimento já cancelado podia ser cancelado novamente. | `Atendimento.cancelar()`. | Adicionada validação do status antes do cancelamento. | Exceções / estados |
| bug08 | Buscar atendimento inexistente podia resultar em `null`. | `AgendaService.buscarPorId()`. | Uso de `orElseThrow()` com `AtendimentoNaoEncontradoException`. | Optional / tratamento de exceções |
| bug09 | A Factory não tratava tipos de atendimento inexistentes corretamente. | `AtendimentoFactory`. | Adicionado `default` no `switch`, lançando `IllegalArgumentException`. | Factory Method |
| bug10 | Builder aceitava atendimento sem nome do pet. | `AtendimentoBuilder.construir()`. | Validação de `petNome` nulo ou vazio. | Builder / validação |
| bug11 | Builder aceitava atendimento sem porte do pet. | `AtendimentoBuilder.construir()`. | Validação de `petPorte` nulo ou vazio. | Builder / encapsulamento |
| bug12 | A duração específica dos tipos de atendimento não era respeitada. | Subclasses de `Atendimento`, especialmente `Banho` e `Tosa`. | Implementação correta de `getDuracaoMinutos()` com `@Override`. | Polimorfismo / sobrescrita |

---

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory` | Parâmetros com nomes pouco descritivos (`p`, `t`, `n`, `po`, `tu`, `d`). | Troquei por nomes claros: `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. |
| clean02 | `AgendaService` | Uso de `System.out.println()` no código de produção. | Removi a impressão do recibo diretamente no service. |
| clean03 | `AtendimentoController` | Método de fidelidade sem utilização na aplicação. | Removi código morto que não participava do fluxo atual da aplicação. |
| clean04 | — | — | Não realizado. |
| clean05 | — | — | Não realizado. |
| clean06 | — | — | Não realizado. |

---

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `AgendaServiceTest.deveRecusarAgendamentoComDataHoraNoPassado` | Não permitir agendamento com data/hora anterior ao momento atual. | Vermelho — revelou ausência da validação de data. |
| teste02 | `AgendaServiceTest.deveRecusarAgendamentoComHorarioJaOcupado` | O mesmo pet não pode possuir dois atendimentos agendados no mesmo horário. | Vermelho — revelou problema na comparação dos valores. |
| teste03 | `AgendaServiceTest.deveRecusarCancelamentoDeAtendimentoJaConcluido` | Atendimento concluído não pode ser cancelado. | Vermelho — revelou falta de validação do status. |
| teste04 | `AgendaServiceTest.deveRecusarCancelamentoDeAtendimentoJaCancelado` | Atendimento cancelado não pode ser cancelado novamente. | Vermelho — revelou falta de validação do status. |
| teste05 | `AgendaServiceTest.deveRecusarConclusaoDeAtendimentoCancelado` | Atendimento cancelado não pode ser concluído. | Vermelho — revelou falta de validação do status. |
| teste06 | `ConsultaVeterinariaTest.deveDurar30Minutos` | Consulta veterinária deve possuir duração de 30 minutos. | Verde — a regra já estava correta. |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)

A suíte de testes funcionou como um contrato porque mostrou exatamente qual comportamento o sistema deveria apresentar. Quando um teste falhava, eu conseguia usar a mensagem do JUnit para descobrir o valor esperado e o valor recebido. Por exemplo, nos testes de agendamento, a comparação de objetos diferentes com o mesmo valor mostrou que a validação do horário precisava comparar o conteúdo e não a referência. Também usei os testes de status para verificar as transições entre `AGENDADO`, `CONCLUIDO` e `CANCELADO`. A vantagem sobre testar tudo manualmente com `curl` é que os testes são repetíveis e automatizados. Depois das correções, toda a suíte pôde ser executada novamente. O resultado final foi de 26 testes sem falhas.

### 2. Mock e injeção de dependência (Aulas 13 a 15)

No `AgendaServiceTest`, o `@Mock` cria uma implementação falsa de `AtendimentoRepository`. O `@InjectMocks` coloca esse objeto falso dentro do `AgendaService`, permitindo testar somente as regras do service. Em produção, quem faz a injeção é o container do Spring por meio do `@Autowired`. No teste, quem faz essa função é o Mockito. Por isso não é necessário iniciar o Spring nem conectar no Oracle para testar o `AgendaService`. Por exemplo, o teste configura `repository.findByPetNome("Rex")` para retornar uma lista e depois verifica se `repository.save()` foi chamado. Dessa maneira, o teste fica isolado do banco de dados.

### 3. `==` vs `.equals()` (Aula 7)

O problema acontece porque `==` compara referências de objetos, enquanto `.equals()` compara o conteúdo quando a classe implementa essa comparação. Isso é importante para `String` e também para `LocalDateTime`. Dois objetos podem representar o mesmo valor sem serem exatamente a mesma referência. Com strings literais como `"Rex"`, o Java pode reutilizar a mesma referência por causa do pool de strings, dando a impressão de que `==` funciona. Isso seria apenas um comportamento circunstancial e não uma comparação segura de conteúdo. Na validação do `AgendaService`, a correção foi usar `.equals()` para comparar o nome do pet e a data/hora. Assim, dois objetos diferentes representando o mesmo valor são tratados corretamente.

### 4. Sobrescrita vs sobrecarga (Aula 7)

Sobrescrita acontece quando uma subclasse implementa novamente um método que já existe na classe pai com a mesma assinatura. No projeto, `Banho` e `Tosa` sobrescrevem `getDuracaoMinutos()` definido em `Atendimento`. Sobrecarga acontece quando existem métodos com o mesmo nome, mas parâmetros diferentes, criando outra assinatura. Um erro de assinatura poderia fazer o método parecer correto, mas o Java trataria como outro método em vez de substituir o comportamento da classe pai. A anotação `@Override` ajuda porque o compilador verifica se realmente existe um método correspondente na superclasse. No projeto, usamos `@Override` em `Banho`, `Tosa` e `ConsultaVeterinaria` para deixar explícito o polimorfismo.

### 5. Singleton manual vs bean do Spring (Aula 14)

O `GeradorProtocolo` utiliza o padrão Singleton para manter uma única instância responsável por gerar os protocolos sequenciais. A classe possui construtor privado e o método `getInstancia()` fornece a instância compartilhada. Dessa forma, chamadas consecutivas usam o mesmo contador. O risco do Singleton manual é que a criação da instância precisa ser controlada pelo próprio código. Já o `AgendaService` utiliza `@Service`, portanto sua instância é gerenciada pelo container do Spring. O Spring controla o ciclo de vida e a injeção de dependências do service. Assim, no `AgendaService`, o código não precisa criar manualmente o repository. A diferença principal é que o Singleton foi implementado manualmente, enquanto o ciclo de vida do service é administrado pelo framework.

### 6. Cobertura de testes: onde parar? (Aula 15)

Os testes que ficaram verdes continuam sendo importantes porque documentam uma regra que deve continuar funcionando no futuro. Um teste verde não é inútil; ele protege o comportamento correto contra alterações futuras. No projeto, por exemplo, o teste da duração da consulta confirma que o comportamento esperado de `30` minutos continua válido. Em um projeto real, eu priorizaria primeiro as regras de negócio mais importantes e os caminhos de erro, além dos caminhos felizes principais. Buscar 100% de cobertura não significa necessariamente testar todos os comportamentos importantes. É melhor ter testes que realmente protejam regras críticas do sistema. A suíte final do projeto ficou com 26 testes e nenhuma falha.

---

## Parte 5 — Espaço livre

A principal dificuldade foi localizar os problemas sem alterar o funcionamento que já estava correto. Os testes ajudaram a identificar as regras de negócio que precisavam ser protegidas, principalmente as validações de agenda e de status. Também foi necessário entender a diferença entre testes unitários com Mockito e testes que dependeriam do Spring ou do banco. Depois das correções, a suíte final apresentou **26 testes, 0 falhas e 0 erros**, indicando que as regras cobertas pelos testes estão funcionando conforme esperado.