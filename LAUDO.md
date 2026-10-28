# Laudo técnico — Operação Resgate: Detetives de Bugs

As correções abaixo foram feitas no código-fonte fornecido. O ambiente não possui a configuração `lib/firebase_options.dart` do Firebase da escola, portanto os testes integrados com autenticação e Firestore precisam ser executados depois de configurar o projeto Firebase correto.

## Bug 01 | Arquivo: `lib/models/ocorrencia.dart`
- **Sintoma observado:** o identificador do responsável podia ser preenchido com o nome/e-mail exibido, fazendo a verificação de propriedade falhar.
- **Hipótese:** `autorId` lia o campo `autorNome` do Firestore.
- **Trecho alterado:** `autorId` agora lê `d['autorId']`.
- **Como testar:** criar uma ocorrência e conferir que `autorId` corresponde ao UID da conta autenticada.

## Bug 02 | Arquivo: `lib/models/ocorrencia.dart`
- **Sintoma observado:** ocorrências abertas apareciam como resolvidas e vice-versa.
- **Hipótese:** o valor booleano `resolvido` era invertido durante a leitura.
- **Trecho alterado:** leitura direta de `d['resolvido']`, mantendo `false` como padrão.
- **Como testar:** cadastrar ocorrência nova e conferir que aparece como “Em aberto”; depois marcar como resolvida.

## Bug 03 | Arquivo: `lib/services/central_service.dart`
- **Sintoma observado:** registros antigos apareciam antes dos novos.
- **Hipótese:** a consulta ordenava `criadoEm` em ordem crescente.
- **Trecho alterado:** `orderBy('criadoEm', descending: true)`.
- **Como testar:** cadastrar duas ocorrências em sequência e conferir a mais recente no topo.

## Bug 04 | Arquivo: `lib/services/central_service.dart`
- **Sintoma observado:** a identificação do responsável não correspondia ao identificador usado pela autenticação.
- **Hipótese:** `autorId` armazenava o e-mail em vez do UID.
- **Trecho alterado:** cadastro agora salva `u.uid` em `autorId`.
- **Como testar:** cadastrar com uma conta Google e comparar o UID autenticado com o campo salvo no Firestore.

## Bug 05 | Arquivo: `lib/services/central_service.dart`
- **Sintoma observado:** a ação de alterar status não alternava o estado atual.
- **Hipótese:** o serviço gravava o mesmo valor que já estava no objeto.
- **Trecho alterado:** atualização grava `!item.resolvido`.
- **Como testar:** marcar como resolvido e depois reabrir, verificando o valor no Firestore.

## Bug 06 | Arquivo: `lib/services/central_service.dart`
- **Sintoma observado:** pistas eram gravadas em um documento fixo, não na ocorrência selecionada.
- **Hipótese:** o caminho usava `doc('pistas')` em vez do ID recebido.
- **Trecho alterado:** subcoleção de pistas agora fica em `ocorrencias/{ocorrenciaId}/pistas`.
- **Como testar:** enviar pistas em duas ocorrências diferentes e verificar se cada uma aparece somente no caso correto.

## Bug 07 | Arquivo: `lib/pages/central_page.dart`
- **Sintoma observado:** o controle “Somente não resolvidos” podia ficar inconsistente com a opção selecionada.
- **Hipótese:** o callback ignorava o novo valor e invertia o estado local.
- **Trecho alterado:** `somenteAbertos = value`.
- **Como testar:** ligar e desligar o controle e conferir o estado visual e a lista.

## Bug 08 | Arquivo: `lib/pages/central_page.dart`
- **Sintoma observado:** o filtro “Somente não resolvidos” mostrava ocorrências resolvidas.
- **Hipótese:** o predicado filtrava `o.resolvido`.
- **Trecho alterado:** filtro agora usa `!o.resolvido`.
- **Como testar:** manter uma ocorrência aberta e outra resolvida e ativar o filtro.

## Bug 09 | Arquivo: `lib/pages/central_page.dart`
- **Sintoma observado:** ao abrir uma ocorrência, a tela de detalhes não recebia os dados selecionados.
- **Hipótese:** `DetalhePage` era criada sem o argumento obrigatório `ocorrencia`.
- **Trecho alterado:** navegação passa `ocorrencia: o`.
- **Como testar:** tocar em ocorrências diferentes e conferir título, local e descrição correspondentes.

## Bug 10 | Arquivo: `lib/pages/central_page.dart`
- **Sintoma observado:** o cadastro podia prosseguir com título ou local vazio.
- **Hipótese:** a validação usava `&&`, exigindo que os dois campos estivessem vazios para bloquear.
- **Trecho alterado:** condição usa `||` para exigir título e local.
- **Como testar:** tentar registrar com título vazio e depois com local vazio; os dois casos devem ser bloqueados.

## Bug 11 | Arquivo: `lib/pages/detalhe_page.dart`
- **Sintoma observado:** o envio de pista era interrompido quando o campo tinha texto.
- **Hipótese:** a condição de retorno tratava texto preenchido como inválido.
- **Trecho alterado:** o método retorna somente quando a pista está vazia.
- **Como testar:** tentar enviar campo vazio e depois uma pista preenchida.

## Bug 12 | Arquivo: `lib/pages/detalhe_page.dart`
- **Sintoma observado:** o serviço recebia uma mensagem vazia mesmo quando havia texto digitado.
- **Hipótese:** a chamada enviava a string literal vazia em vez do conteúdo do campo.
- **Trecho alterado:** chamada usa `pista.text.trim()` e limpa o campo após o envio.
- **Como testar:** enviar uma pista e confirmar que a mensagem e o autor aparecem na ocorrência certa.

## Verificações finais pendentes
- [ ] Executar `flutter pub get` e `flutter analyze`.
- [ ] Adicionar o arquivo `lib/firebase_options.dart` gerado pelo FlutterFire para o projeto autorizado da escola.
- [ ] Confirmar login Google, regras de segurança do Firestore e sincronização ao vivo em duas contas.
- [ ] Fazer uma captura real das duas contas vendo a mesma ocorrência e anexá-la à entrega.

**Observação:** não foram removidos recursos, autenticação ou verificações de autorização. A captura de duas contas não foi fabricada; ela deve ser feita após a configuração e o teste real do Firebase.
