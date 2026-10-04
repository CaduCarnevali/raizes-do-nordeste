# Raízes do Nordeste

Protótipo acadêmico responsivo para pedidos e retirada em uma rede fictícia de lanchonetes.

**Aplicação publicada:** https://caducarnevali.github.io/raizes-do-nordeste/

**Aluno:** Henrique da Silva Carnevali  
**RU:** 4390921

## Tecnologias

- Vue 3
- Vite
- Bootstrap 5.3
- Dados mockados em JSON

Não existe back-end, banco de dados ou integração real de pagamento. As interações permanecem apenas na memória durante a navegação e são reiniciadas quando a página é recarregada.

## Executar localmente

```bash
npm install
npm run dev
```

## Gerar versão de publicação

```bash
npm run build
```

Os arquivos estáticos serão gerados na pasta `dist`.

## Usuário inicial

```text
E-mail: cliente@raizes.com.br
Senha: raizes123
```

Outras contas para validar saldos diferentes:

```text
joao@raizes.com.br / nordeste123 — 15 pontos
ana@raizes.com.br / clube123 — 120 pontos
```

Contas criadas pela interface podem sair e entrar novamente enquanto a página permanecer aberta. Ao recarregar, a lista volta aos dados iniciais do JSON.

O consentimento de privacidade e a preferência de promoções são solicitados uma única vez por conta durante a sessão. Depois de aceitos, os próximos pedidos da mesma conta avançam diretamente para o pagamento.

## Clube Raízes

- Cada R$ 1,00 efetivamente pago gera 1 ponto.
- Cada ponto vale R$ 0,05 de desconto, equivalente a 5% do valor que originou o ponto.
- Ao aplicar o saldo, o sistema usa o máximo possível sem ultrapassar o valor do pedido.
- Pontos e descontos são mantidos somente em memória durante a sessão.

Para os testes de pagamento, `4111 1111 1111 1111` é aprovado e qualquer número terminado em `0000` é recusado.
