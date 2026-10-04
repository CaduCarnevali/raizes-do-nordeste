# Plano de Testes da Interface

## 1. Objetivo

Este plano define a estratégia de validação da aplicação web Raízes do Nordeste. Os testes verificam os fluxos principais do pedido, a clareza da interface, a adaptação a diferentes tamanhos de tela e o atendimento aos requisitos de privacidade e consentimento. Como o projeto utiliza dados mockados e não possui back-end, banco de dados ou pagamento real, as validações consideram o comportamento da aplicação durante uma sessão do navegador.

## 2. Estratégia de testes

Serão realizados testes funcionais manuais, testes de usabilidade, validações responsivas e verificações técnicas de compilação. Cada cenário deve ser executado a partir de um estado conhecido, com registro do resultado obtido e, quando útil, captura de tela.

Os testes estão organizados nas seguintes categorias:

- Fluxo principal: unidade, cardápio, carrinho, identificação, privacidade, pagamento e acompanhamento.
- Fluxos alternativos e negativos: autenticação inválida, cadastro inconsistente, pagamento recusado e ações sem pré-condições.
- Fidelização: geração de pontos, conversão em crédito e limite do desconto.
- Privacidade: consentimento obrigatório, preferência opcional e aceite único por conta durante a sessão.
- Responsividade e usabilidade: celular, desktop e totem.
- Persistência simulada: manutenção dos dados em memória e restauração após recarregar a página.

## 3. Ambiente e dados de teste

### 3.1 Ambiente

- Navegador principal: Google Chrome ou Microsoft Edge atualizado.
- Execução local: `npm run dev`.
- Validação de produção: `npm run build` e `npm run preview`.
- Conexão: não é necessária após o carregamento local dos arquivos do projeto.

### 3.2 Resoluções de referência

| Canal | Resolução de referência | Forma de interação |
|---|---:|---|
| Celular | 390 × 844 px | Toque e rolagem vertical |
| Desktop | 1366 × 768 px | Mouse e teclado |
| Totem | 1080 × 1920 px | Toque, orientação vertical |

### 3.3 Contas mockadas

| Perfil | E-mail | Senha | Saldo inicial |
|---|---|---|---:|
| Marina Oliveira | cliente@raizes.com.br | raizes123 | 68 pontos |
| João Santos | joao@raizes.com.br | nordeste123 | 15 pontos |
| Ana Costa | ana@raizes.com.br | clube123 | 120 pontos |

### 3.4 Dados de pagamento

- Pagamento aprovado: `4111 1111 1111 1111`.
- Pagamento recusado: qualquer número cujo final seja `0000`.
- Validade válida para teste: `12/30`.
- CVV válido para teste: `123`.

### 3.5 Regra do programa de pontos

- Cada R$ 1,00 efetivamente pago gera 1 ponto inteiro.
- Cada ponto vale R$ 0,05 de desconto.
- A aplicação utiliza o menor limite entre o saldo de pontos e a quantidade necessária para zerar o pedido.
- Os pontos são descontados apenas depois da aprovação do pagamento.

## 4. Critérios de aceitação

O fluxo será considerado aprovado quando:

- todas as funções essenciais puderem ser concluídas sem erro de execução;
- mensagens de erro forem claras e apresentadas próximas da ação correspondente;
- nenhuma etapa obrigatória puder ser ignorada indevidamente;
- os totais do carrinho, descontos, pontos e histórico permanecerem coerentes;
- o conteúdo continuar legível e operável nas três resoluções de referência;
- o consentimento obrigatório e a preferência opcional forem tratados separadamente;
- o build de produção for concluído sem erros.

## 5. Cenários de teste

### CT 01 Seleção da unidade e carregamento do cardápio

**Objetivo:** verificar se a escolha de uma unidade disponibiliza o cardápio correspondente.

**Pré-condição:** aplicação recém-aberta, sem unidade selecionada.

**Entrada ou ação:** selecionar Unidade Centro.

**Passos:**

1. Abrir a aplicação.
2. Localizar a seção de unidades.
3. Selecionar Unidade Centro.
4. Observar o resumo da unidade e os produtos exibidos.

**Resultado esperado:** a unidade selecionada aparece no cabeçalho e no resumo, o cardápio da Unidade Centro é carregado e os controles do pedido são habilitados.

**Validações:** não deve ser possível abrir um pedido antes de selecionar uma unidade; nome, endereço e tempo de preparo devem estar legíveis.

### CT 02 Troca de unidade com itens no carrinho

**Objetivo:** garantir que produtos de unidades diferentes não sejam misturados.

**Pré-condição:** Unidade Centro selecionada e pelo menos um item no carrinho.

**Entrada ou ação:** trocar para Unidade Shopping.

**Passos:**

1. Adicionar um produto ao pedido.
2. Acionar Trocar unidade.
3. Selecionar Unidade Shopping.
4. Abrir o carrinho.

**Resultado esperado:** o carrinho anterior é limpo, o cardápio é atualizado e uma mensagem informa que a unidade foi alterada.

**Validações:** nenhum produto da unidade anterior deve permanecer no pedido.

### CT 03 Inclusão e alteração de itens no carrinho

**Objetivo:** validar inclusão, aumento, redução e remoção de produtos.

**Pré-condição:** unidade selecionada e cardápio carregado.

**Entrada ou ação:** adicionar um produto e alterar sua quantidade.

**Passos:**

1. Adicionar um produto ao carrinho.
2. Abrir Pedido.
3. Aumentar a quantidade.
4. Reduzir a quantidade até zero.

**Resultado esperado:** quantidade, contador e subtotal são recalculados; ao chegar a zero, o item é removido.

**Validações:** o subtotal deve ser igual à soma do preço multiplicado pela quantidade de cada item.

### CT 04 Inclusão da promoção

**Objetivo:** verificar se a oferta semanal pode ser adicionada e possui efeito real no pedido.

**Pré-condição:** unidade compatível selecionada.

**Entrada ou ação:** clicar em Adicionar combo.

**Passos:**

1. Localizar a oferta semanal.
2. Adicionar o combo.
3. Abrir o carrinho.

**Resultado esperado:** o combo aparece como um item único com preço promocional, descrição e subtotal corretos.

**Validações:** o valor original não deve ser somado ao carrinho.

### CT 05 Login com credenciais válidas

**Objetivo:** validar a autenticação mockada.

**Pré-condição:** item no carrinho e usuário desconectado.

**Entrada ou ação:** `cliente@raizes.com.br` e `raizes123`.

**Passos:**

1. Iniciar a finalização do pedido.
2. Preencher e-mail e senha válidos.
3. Clicar em Entrar e continuar.

**Resultado esperado:** o usuário Marina é identificado, seu saldo aparece no cabeçalho e o fluxo avança para privacidade ou pagamento, conforme o consentimento da conta.

**Validações:** a senha não deve aparecer no perfil nem no histórico.

### CT 06 Login com credenciais inválidas

**Objetivo:** validar o tratamento de falha de autenticação.

**Pré-condição:** tela de identificação aberta.

**Entrada ou ação:** e-mail válido com senha incorreta.

**Passos:**

1. Informar `cliente@raizes.com.br`.
2. Informar uma senha diferente de `raizes123`.
3. Tentar entrar.

**Resultado esperado:** o acesso não é liberado e a mensagem de e-mail ou senha incorretos é exibida.

**Validações:** o usuário deve permanecer na etapa de identificação e o carrinho deve ser preservado.

### CT 07 Cadastro com dados inconsistentes

**Objetivo:** verificar as validações do formulário de cadastro.

**Pré-condição:** tela de identificação aberta na opção Criar conta.

**Entrada ou ação:** e-mail já existente, senha curta ou confirmação divergente.

**Passos:**

1. Tentar cadastrar um e-mail já utilizado.
2. Corrigir o e-mail e informar senha com menos de seis caracteres.
3. Informar senhas válidas, mas diferentes entre si.

**Resultado esperado:** cada tentativa é impedida com mensagem específica; nenhuma conta inválida é criada.

**Validações:** o formulário deve manter os demais dados para permitir correção.

### CT 08 Cadastro válido e reutilização da conta na sessão

**Objetivo:** validar a criação de conta mockada e um novo login durante a mesma sessão.

**Pré-condição:** e-mail ainda não utilizado na sessão.

**Entrada ou ação:** nome, e-mail válido e duas senhas idênticas com seis ou mais caracteres.

**Passos:**

1. Criar a conta.
2. Confirmar que o usuário foi autenticado.
3. Sair da conta.
4. Entrar novamente com as credenciais criadas.

**Resultado esperado:** a conta é criada com zero pontos e pode ser reutilizada até a página ser recarregada.

**Validações:** após F5, a conta criada deve deixar de existir, conforme a estratégia de dados mockados.

### CT 09 Bloqueio sem consentimento obrigatório

**Objetivo:** garantir que o pedido não avance sem o aceite necessário.

**Pré-condição:** conta autenticada sem consentimento registrado.

**Entrada ou ação:** manter desmarcado o aceite obrigatório.

**Passos:**

1. Abrir a etapa de privacidade.
2. Não marcar o aceite obrigatório.
3. Observar o botão de continuação.

**Resultado esperado:** o botão permanece desabilitado e o pagamento não pode ser acessado.

**Validações:** a finalidade do uso de nome e e-mail deve estar visível antes do aceite.

### CT 10 Consentimento único por conta

**Objetivo:** confirmar que o aviso obrigatório é aceito apenas uma vez por conta durante a sessão.

**Pré-condição:** conta autenticada sem consentimento registrado.

**Entrada ou ação:** aceitar o aviso e concluir a etapa.

**Passos:**

1. Marcar o consentimento obrigatório.
2. Clicar em Continuar para pagamento.
3. Concluir ou iniciar outro pedido.
4. Sair e entrar novamente com a mesma conta sem recarregar a página.
5. Continuar o novo pedido.

**Resultado esperado:** a primeira confirmação fica associada à conta e os pedidos seguintes avançam diretamente para o pagamento.

**Validações:** somente marcar a caixa sem clicar em Continuar para pagamento não deve registrar o consentimento.

### CT 11 Recusa da preferência opcional de promoções

**Objetivo:** verificar a separação entre consentimento necessário e marketing opcional.

**Pré-condição:** etapa de privacidade aberta.

**Entrada ou ação:** aceitar o aviso obrigatório e recusar promoções.

**Passos:**

1. Marcar apenas o aceite obrigatório.
2. Manter desmarcada a opção de promoções.
3. Continuar.

**Resultado esperado:** o pagamento é liberado normalmente e a preferência de marketing fica registrada como falsa.

**Validações:** a recusa de promoções não pode impedir a realização do pedido.

### CT 12 Aplicação de pontos como desconto

**Objetivo:** validar a conversão de pontos em crédito de 5%.

**Pré-condição:** login de Marina, saldo de 68 pontos e pedido de R$ 89,70.

**Entrada ou ação:** clicar em Usar pontos.

**Passos:**

1. Avançar até o pagamento.
2. Conferir o saldo e o valor do crédito.
3. Clicar em Usar pontos.
4. Conferir o resumo financeiro.

**Resultado esperado:** são aplicados 68 pontos, equivalentes a R$ 3,40, e o total a pagar passa para R$ 86,30.

**Validações:** subtotal, quantidade de pontos, desconto e total devem aparecer separadamente; remover os pontos deve restaurar o total original.

### CT 13 Limite do desconto por pontos

**Objetivo:** garantir que o desconto nunca ultrapasse o saldo nem produza total negativo.

**Pré-condição:** conta com saldo de pontos suficiente para cobrir integralmente um pedido.

**Entrada ou ação:** aplicar os pontos no pagamento.

**Passos:**

1. Montar um pedido de valor inferior ao crédito disponível.
2. Aplicar os pontos.
3. Confirmar o pedido.

**Resultado esperado:** apenas os pontos necessários são utilizados, o total chega exatamente a R$ 0,00 e os pontos excedentes permanecem na conta.

**Validações:** os campos de cartão deixam de ser necessários e nenhum valor negativo é apresentado.

### CT 14 Pagamento aprovado

**Objetivo:** validar o retorno positivo do pagamento simulado.

**Pré-condição:** pedido com valor maior que zero na etapa de pagamento.

**Entrada ou ação:** cartão `4111 1111 1111 1111`, nome, validade `12/30` e CVV `123`.

**Passos:**

1. Preencher os dados do cartão.
2. Confirmar o pagamento.
3. Aguardar a transição.

**Resultado esperado:** é exibida a confirmação, um número de pedido é gerado e a tela avança para o acompanhamento.

**Validações:** o pedido deve ser adicionado ao histórico com unidade, itens, pontos utilizados, total pago e status inicial.

### CT 15 Pagamento recusado

**Objetivo:** validar o retorno negativo do pagamento simulado.

**Pré-condição:** etapa de pagamento aberta com valor maior que zero.

**Entrada ou ação:** cartão terminado em `0000`.

**Passos:**

1. Informar os dados obrigatórios.
2. Usar um número terminado em `0000`.
3. Tentar pagar.
4. Acionar Usar outro cartão.

**Resultado esperado:** a compra não é concluída, uma mensagem clara de recusa é exibida e o usuário pode tentar novamente.

**Validações:** pontos não são descontados, novos pontos não são concedidos e nenhum pedido é incluído no histórico.

### CT 16 Continuidade da etapa pelo carrinho

**Objetivo:** verificar se Continuar pedido retorna ao ponto em que o cliente parou.

**Pré-condição:** checkout já iniciado na etapa de privacidade ou pagamento.

**Entrada ou ação:** abrir Pedido no cabeçalho e clicar em Continuar pedido.

**Passos:**

1. Avançar até o pagamento.
2. Rolar para o início da página.
3. Abrir o painel Pedido.
4. Clicar em Continuar pedido.

**Resultado esperado:** o painel fecha e a página rola até a etapa de pagamento, sem reiniciar o fluxo.

**Validações:** campos já preenchidos, consentimento e desconto aplicado devem ser preservados.

### CT 17 Atualização do status e histórico

**Objetivo:** validar o ciclo de acompanhamento do pedido.

**Pré-condição:** pagamento aprovado.

**Entrada ou ação:** clicar sucessivamente em Atualizar status.

**Passos:**

1. Conferir o status Pedido recebido.
2. Atualizar para Em preparo.
3. Atualizar para Pronto para retirada.
4. Atualizar para Pedido retirado.
5. Abrir Meus pedidos.

**Resultado esperado:** a linha do tempo avança até a conclusão e o mesmo status aparece no histórico.

**Validações:** o pedido deve manter número, data, unidade, endereço, itens, totais e pontos.

### CT 18 Novo pedido preservando conta e saldo

**Objetivo:** verificar o reinício do ciclo sem perder os dados da conta.

**Pré-condição:** pedido marcado como retirado.

**Entrada ou ação:** clicar em Fazer novo pedido.

**Passos:**

1. Concluir o acompanhamento.
2. Iniciar um novo pedido.
3. Adicionar produtos.
4. Continuar o checkout.

**Resultado esperado:** o carrinho anterior é limpo, a unidade e a conta permanecem ativas, o saldo atualizado é preservado e o consentimento não é solicitado novamente.

**Validações:** o novo pedido não pode alterar os dados do pedido anterior no histórico.

### CT 19 Responsividade em celular

**Objetivo:** validar legibilidade e operação em tela pequena.

**Pré-condição:** DevTools configurado em 390 × 844 px.

**Entrada ou ação:** executar o fluxo principal usando a visualização móvel.

**Passos:**

1. Selecionar unidade.
2. Navegar pelas categorias e adicionar itens.
3. Abrir a barra fixa do pedido.
4. Fazer login, aceitar a privacidade e acessar o pagamento.

**Resultado esperado:** não há rolagem horizontal, cortes ou sobreposição; botões e campos são utilizáveis por toque; o carrinho móvel permanece acessível.

**Validações:** textos, preços, mensagens de erro e controles devem permanecer legíveis.

### CT 20 Responsividade em desktop e totem

**Objetivo:** verificar a adaptação do layout aos canais Web e Totem.

**Pré-condição:** DevTools nas resoluções 1366 × 768 e 1080 × 1920 px.

**Entrada ou ação:** percorrer unidade, cardápio, carrinho e pagamento nas duas resoluções.

**Passos:**

1. Testar o fluxo em desktop.
2. Repetir em orientação vertical de totem.
3. Verificar áreas clicáveis, colunas, cartões e formulários.

**Resultado esperado:** no desktop, o espaço horizontal é bem aproveitado; no totem, controles possuem tamanho confortável para toque e o conteúdo segue uma ordem vertical clara.

**Validações:** nenhum componente deve ficar inacessível, cortado ou sobreposto.

### CT 21 Restauração dos mocks após recarregar

**Objetivo:** comprovar o comportamento temporário dos dados mockados.

**Pré-condição:** conta criada ou alterada, pedido realizado e saldo modificado na sessão.

**Entrada ou ação:** atualizar a página com F5.

**Passos:**

1. Registrar uma conta ou concluir um pedido.
2. Confirmar a alteração em memória.
3. Recarregar a página.
4. Entrar com uma conta original do arquivo JSON.

**Resultado esperado:** as alterações da sessão desaparecem e os dados retornam ao estado definido nos arquivos JSON.

**Validações:** a aplicação não deve afirmar que os dados são persistidos permanentemente.

### CT 22 Build de produção

**Objetivo:** garantir que a aplicação possa ser publicada como site estático.

**Pré-condição:** dependências instaladas.

**Entrada ou ação:** executar `npm run build`.

**Passos:**

1. Abrir o terminal na pasta do projeto.
2. Executar o comando de build.
3. Verificar a saída do processo e a pasta `dist`.

**Resultado esperado:** o Vite conclui a compilação sem erros e gera os arquivos estáticos de produção.

**Validações:** `dist/index.html` e os arquivos de recursos devem existir.

## 6. Registro da execução

Durante a execução, preencher a tabela abaixo e guardar evidências dos cenários mais importantes.

| Cenário | Data | Navegador ou resolução | Resultado | Evidência | Observação |
|---|---|---|---|---|---|
| CT 01 | 04/10/2026 | Desktop | Aprovado | 02_cardapio_desktop.png | Unidade Centro carregou 13 produtos. |
| CT 02 | 04/10/2026 | Desktop | Aprovado | Registro da execução | Unidade Shopping carregou 12 produtos e limpou o carrinho. |
| CT 03 | 04/10/2026 | Desktop | Aprovado | 03_carrinho_promocao.png | Quantidade e remoção em zero validadas. |
| CT 04 | 04/10/2026 | Desktop | Aprovado | 03_carrinho_promocao.png | Combo incluído por R$ 46,90. |
| CT 05 | 04/10/2026 | Desktop | Aprovado | 05_privacidade_lgpd.png | Conta Marina autenticada com saldo carregado. |
| CT 06 | 04/10/2026 | Desktop | Aprovado | 04_login_invalido.png | Mensagem de credenciais incorretas exibida. |
| CT 07 | 04/10/2026 | Desktop | Aprovado | 12_cadastro_valido.png | E-mail duplicado, senha curta e confirmação divergente foram bloqueados. |
| CT 08 | 04/10/2026 | Desktop | Aprovado | 12_cadastro_valido.png | Conta criada e reutilizada na mesma sessão. |
| CT 09 | 04/10/2026 | Desktop | Aprovado | 05_privacidade_lgpd.png | Botão permaneceu desabilitado sem aceite. |
| CT 10 | 04/10/2026 | Desktop | Aprovado | Registro da execução | Segundo pedido avançou diretamente ao pagamento. |
| CT 11 | 04/10/2026 | Desktop | Aprovado | 05_privacidade_lgpd.png | Marketing desmarcado não bloqueou o avanço. |
| CT 12 | 04/10/2026 | Desktop | Aprovado | 06_pagamento_pontos.png | 68 pontos geraram R$ 3,40 de desconto. |
| CT 13 | 04/10/2026 | Desktop | Aprovado | 14_pontos_zerando_pedido.png | 158 de 162 pontos zeraram R$ 7,90; 4 pontos permaneceram. |
| CT 14 | 04/10/2026 | Desktop | Aprovado | 08_acompanhamento.png | Pedido e pontos foram gerados após aprovação. |
| CT 15 | 04/10/2026 | Desktop | Aprovado | 07_pagamento_recusado.png | Recusa exibida sem registrar o pedido. |
| CT 16 | 04/10/2026 | Desktop | Aprovado | 13_continuar_etapa_pagamento.png | Carrinho retornou à etapa de pagamento após correção do fallback de rolagem. |
| CT 17 | 04/10/2026 | Desktop | Aprovado | 09_historico_pedidos.png | Status final e histórico permaneceram coerentes. |
| CT 18 | 04/10/2026 | Desktop | Aprovado | Registro da execução | Conta, pontos e consentimento foram preservados. |
| CT 19 | 04/10/2026 | 390 × 844 px | Aprovado | 10_mobile_390x844.png | Sem rolagem horizontal. |
| CT 20 | 04/10/2026 | Desktop e 1080 × 1920 px | Aprovado | 01_inicio_desktop.png e 11_totem_1080x1920.png | Sem rolagem horizontal. |
| CT 21 | 04/10/2026 | Desktop | Aprovado | Registro da execução | F5 restaurou os mocks originais. |
| CT 22 | 04/10/2026 | Terminal | Aprovado | Saída do build | Vite concluiu a compilação sem erros. |

## 7. Evidências recomendadas

Para o relatório final, recomenda-se registrar ao menos:

1. seleção da unidade e cardápio carregado;
2. carrinho com itens e promoção;
3. login válido e mensagem de login inválido;
4. aviso de privacidade com consentimento obrigatório;
5. resumo do pagamento com desconto de pontos;
6. pagamento recusado;
7. pagamento aprovado e linha do tempo;
8. histórico contendo unidade e valores;
9. visualização em celular;
10. visualização vertical de totem;
11. terminal com o build concluído.

## 8. Resultado esperado da validação

A aplicação será considerada pronta para publicação quando os cenários críticos CT 01, CT 03, CT 05, CT 09, CT 10, CT 12, CT 14, CT 15, CT 17, CT 19, CT 20 e CT 22 estiverem aprovados e não houver defeitos que impeçam a conclusão do pedido.

## 9. Resultado consolidado

Os 22 cenários foram executados e aprovados em 04/10/2026. O CT 16 identificou uma condição de temporização no fechamento do painel do carrinho; foi acrescentado um fallback de rolagem e o cenário passou na repetição. Depois da correção, o build de produção também foi executado novamente e concluído sem erros.
