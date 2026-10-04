# Matriz de Requisitos e Evidências

## 1. Finalidade

Esta matriz relaciona os requisitos do Projeto Multidisciplinar Front End às funcionalidades implementadas na aplicação Raízes do Nordeste e às evidências que deverão compor o relatório final. A matriz permite verificar a cobertura do sistema e identificar os itens documentais ainda pendentes.

## 2. Requisitos funcionais

| ID | Requisito | Atendimento | Implementação ou evidência prevista |
|---|---|---|---|
| RF 01 | Permitir cadastro de usuário | Atendido | Formulário Criar conta com validação de e-mail duplicado, tamanho da senha e confirmação. |
| RF 02 | Permitir autenticação de usuário | Atendido | Login com contas carregadas de `public/data/customers.json` e contas criadas na sessão. |
| RF 03 | Exibir cardápio de acordo com a unidade | Atendido | Seleção entre Unidade Centro e Unidade Shopping, com filtro pela propriedade `units` dos produtos. |
| RF 04 | Permitir montagem e revisão do pedido | Atendido | Inclusão de produtos e combos, alteração de quantidade, remoção e cálculo do subtotal. |
| RF 05 | Permitir realização do pedido | Atendido | Fluxo guiado por identificação, privacidade, pagamento, confirmação e acompanhamento. |
| RF 06 | Solicitar pagamento por serviço externo | Atendido por simulação | Formulário de cartão representa a integração externa sem transmitir dados reais. |
| RF 07 | Informar pagamento aprovado ou recusado | Atendido | Cartão padrão gera aprovação; números terminados em `0000` geram recusa e nova tentativa. |
| RF 08 | Permitir acompanhamento do pedido | Atendido | Linha do tempo com Pedido recebido, Em preparo, Pronto para retirada e Pedido retirado. |
| RF 09 | Oferecer programa de fidelização | Atendido | Cada R$ 1 pago gera 1 ponto; cada ponto vale R$ 0,05 de desconto em pedido futuro. |
| RF 10 | Apresentar promoções e campanhas | Atendido | Oferta semanal com preço promocional e inclusão funcional no carrinho. |
| RF 11 | Manter histórico de pedidos da conta | Atendido | Painel Meus pedidos com número, data, unidade, status, itens, pontos e valores. |
| RF 12 | Registrar a unidade de retirada | Atendido | Cada pedido guarda nome e endereço da unidade selecionada. |
| RF 13 | Manter o progresso do checkout | Atendido | Continuar pedido retorna à etapa atual sem reiniciar o fluxo. |
| RF 14 | Registrar consentimento de privacidade | Atendido | Aceite obrigatório salvo uma vez por conta durante a sessão, com data da confirmação. |
| RF 15 | Permitir recusa de comunicações promocionais | Atendido | Preferência de marketing é opcional e não bloqueia o pedido. |

## 3. Requisitos não funcionais

| ID | Requisito | Atendimento | Implementação ou evidência prevista |
|---|---|---|---|
| RNF 01 | Interface responsiva | Atendido | Bootstrap 5.3, grade fluida e regras CSS para celular, desktop e telas verticais. |
| RNF 02 | Abordagem mobile first | Atendido | Carrinho móvel fixo, formulários em uma coluna e controles adaptados a telas pequenas. |
| RNF 03 | Usabilidade e clareza | Atendido | Etapas visíveis, mensagens de retorno, estados vazios, totais detalhados e ações nomeadas. |
| RNF 04 | Boa performance | Atendido | Aplicação estática, dados JSON locais e build otimizado pelo Vite. |
| RNF 05 | Escalabilidade da interface | Atendido no escopo | Cardápio e unidades externos ao componente, carregados de arquivos JSON. |
| RNF 06 | Considerar App, Web e Totem | Atendido na interface | Mesmo fluxo responsivo, com validações previstas para 390 × 844, 1366 × 768 e 1080 × 1920 px. |
| RNF 07 | Privacidade e minimização de dados | Atendido | Cadastro solicita apenas nome, e-mail e senha; finalidade é explicada antes do consentimento. |
| RNF 08 | Acessibilidade básica | Parcialmente atendido | Rótulos associados a campos, textos alternativos para controles, navegação semântica e estados de alerta. Auditoria final ainda pendente. |
| RNF 09 | Compatibilidade com publicação estática | Atendido | Build do Vite gera a pasta `dist` sem necessidade de servidor de aplicação. |
| RNF 10 | Persistência coerente com dados mockados | Atendido | Dados permanecem em memória durante a sessão e retornam ao JSON após recarregar a página. |

## 4. LGPD e privacidade

| Verificação | Atendimento | Evidência prevista |
|---|---|---|
| Aviso de privacidade visível na interface | Atendido | Captura da etapa Suas escolhas de privacidade. |
| Finalidade do uso de dados explicada | Atendido | Texto informa identificação, confirmação, retirada e fidelidade. |
| Consentimento obrigatório explícito | Atendido | Caixa de seleção obrigatória e botão desabilitado sem aceite. |
| Marketing tratado separadamente | Atendido | Caixa opcional com informação de que a recusa não impede o pedido. |
| Consentimento vinculado à conta | Atendido | Próximos pedidos da mesma conta ignoram nova solicitação durante a sessão. |
| Coleta limitada ao necessário | Atendido no protótipo | Nome e e-mail identificam a conta; senha é usada apenas na autenticação mockada. |
| Persistência permanente ou transmissão de dados | Não aplicável | O protótipo não possui banco, API de clientes ou envio de dados. |

## 5. Artefatos obrigatórios da entrega

| Artefato | Situação | Próxima ação |
|---|---|---|
| Aplicação codificada | Concluída | Build de produção e 22 cenários aprovados. |
| Dados mockados | Concluídos | Manter os arquivos JSON junto ao projeto. |
| Plano com pelo menos 10 testes | Concluído | Vinte e dois cenários executados e aprovados. |
| Análise do problema | Concluída | Inserida na seção 2.1 do relatório. |
| Requisitos funcionais e não funcionais | Concluídos | Inseridos nas seções 2.2 e 2.3 do relatório. |
| Diagrama de casos de uso | Concluído | Produzido com os cinco atores exigidos. |
| Descrição detalhada de funcionalidade | Concluída | Fluxo Realizar pedido documentado. |
| Jornada do usuário | Concluída | Representada da escolha da unidade até a retirada. |
| Wireframe mobile | Concluído | Inserido na seção 4 do relatório. |
| Wireframe desktop | Concluído | Inserido na seção 4 do relatório. |
| Consideração do canal totem | Concluída | Wireframe e evidência em 1080 × 1920 px. |
| Seção de LGPD | Concluída | Requisitos e evidência incluídos no relatório. |
| Capturas da aplicação | Concluídas | Evidências funcionais e responsivas registradas. |
| Repositório Git público | Pendente | Criar ou publicar e testar em janela anônima. |
| URL pública da aplicação | Pendente | Publicar e testar em janela anônima. |
| Declaração de uso de IA | Concluída | Inserida na seção 10 do relatório. |
| PDF único para o AVA | Pendente | Gerar após concluir os itens anteriores. |

## 6. Critério de prontidão para a entrega

O projeto estará pronto para envio quando os cenários críticos do plano de testes estiverem aprovados, as evidências visuais estiverem inseridas no relatório, os diagramas e wireframes estiverem concluídos e os dois links públicos funcionarem em uma janela anônima. O arquivo final deverá seguir o padrão `4390921_Projeto_Front_End.pdf`.
