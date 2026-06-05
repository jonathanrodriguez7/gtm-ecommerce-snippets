# 🛒 GTM E-commerce Snippets

> Coleção de tags HTML personalizadas para Google Tag Manager com implementação de DataLayer GA4 e Dynamic Remarketing para plataformas de e-commerce brasileiras.

![GTM](https://img.shields.io/badge/Google_Tag_Manager-246FDB?style=flat&logo=googletagmanager&logoColor=white)
![GA4](https://img.shields.io/badge/GA4-E37400?style=flat&logo=googleanalytics&logoColor=white)
![Google Ads](https://img.shields.io/badge/Google_Ads-4285F4?style=flat&logo=googleads&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=flat)

---

## 📦 Plataformas suportadas

| Arquivo | Plataforma | Tipo | Eventos cobertos |
|---|---|---|---|
| `irroba.html` | Irroba | GA4 DataLayer | view_item, add_to_cart, view_cart, begin_checkout, purchase |
| `linx.html` | Linx Commerce | GA4 DataLayer | view_item, add_to_cart, view_cart, begin_checkout, add_shipping_info, add_payment_info, purchase |
| `vtexdynamicrmkt.html` | VTEX Legacy | Dynamic Remarketing | view_item, view_item_list, add_to_cart, purchase |

---

## 🚀 Como usar

Todos os snippets são **tags de HTML Personalizado** do GTM. Para implementar:

1. Acesse seu container no [Google Tag Manager](https://tagmanager.google.com)git remote remove origin

2. Crie uma nova tag → **HTML Personalizado**
3. Cole o conteúdo do arquivo `.html` correspondente
4. Configure o trigger adequado (ver seção de cada snippet abaixo)
5. Publique o container

---

## 📄 Irroba — GA4 DataLayer

**Arquivo:** `irroba.html`

Detecta a página atual via `document.body.classList` e dispara os eventos GA4 correspondentes lendo o objeto `ecommerce gtm` nativo da plataforma Irroba.

### Pré-requisitos
- Tag GA4 configurada no mesmo container
- Nenhuma variável GTM adicional necessária

### Eventos e Triggers

| Evento GA4 | Página detectada | Como detecta |
|---|---|---|
| `view_item` | Página de produto | `body.classList` → `page-product-product` |
| `add_to_cart` | Página de produto | Click em `#button-cart` |
| `view_cart` | Carrinho | `body.classList` → `page-checkout-cart` |
| `begin_checkout` | Carrinho | Click em `a[href="checkout"]` |
| `purchase` | Confirmação | `body.classList` → `page-checkout-success` |

### Trigger GTM sugerido
- **Tipo:** Todas as páginas (`All Pages`)
- O script detecta internamente a página via classList

### Exemplo de dataLayer gerado
```js
// view_item
{
  event: 'view_item',
  ecommerce: {
    items: [{
      item_id: '12345',
      item_name: 'Camiseta Polo',
      price: 99.90,
      item_brand: 'Marca X',
      item_category: 'Camisetas',
      variant: '',
      quantity: 1,
      currency: 'BRL'
    }]
  }
}

// purchase
{
  event: 'purchase',
  ecommerce: {
    transaction_id: 'ORD-001',
    affiliation: 'Loja',
    value: 199.80,
    shipping: 15.00,
    currency: 'BRL',
    coupon: '',
    items: [{ ... }]
  }
}
```

---

## 📄 Linx Commerce — GA4 DataLayer

**Arquivo:** `linx.html`

Integração com o objeto nativo `EzGaCfg` da plataforma Linx. Suporta checkout baseado em **hash routing** (SPA), com listener de `hashchange` para capturar navegação entre etapas sem reload de página.

### Pré-requisitos
- Objeto `EzGaCfg` disponível na página (nativo da Linx)
- Tag GA4 configurada no mesmo container

### Eventos e Triggers

| Evento GA4 | Página / Condição |
|---|---|
| `view_item` | `body.classList` contém `context-product` |
| `add_to_cart` | Click em botão com texto `COMPRAR` |
| `view_cart` | `pathname === '/carrinho'` |
| `begin_checkout` | `hash === '#delivery'` |
| `add_shipping_info` | `hash === '#delivery'` + opção de entrega selecionada |
| `add_payment_info` | Click em `#form-checkout-submit` |
| `purchase` | `hash === '#confirmation'` |

### Trigger GTM sugerido
- **Tipo:** Todas as páginas (`All Pages`)
- O script usa IIFE e trata hash routing internamente

### Observação sobre preços
A função `g_strToNum` divide por `100` (formato centavos padrão: `R$ 99,90` → `9990 / 100 = 99.90`). Verifique o formato retornado pela sua loja antes de usar.

---

## 📄 VTEX Legacy — Dynamic Remarketing (Google Ads)

**Arquivo:** `vtexdynamicrmkt.html`

Tag de **Remarketing Dinâmico** para Google Ads na plataforma VTEX Legacy. Utiliza a variável GTM `{{pageCategory}}` para identificar o contexto da página e disparar os eventos correspondentes.

> ⚠️ Este snippet **não substitui** a tag GA4 de e-commerce. Ele complementa, convertendo os dados para o formato de remarketing do Google Ads.

### Pré-requisitos

**Variável GTM obrigatória:**

| Nome da variável | Tipo | Chave do dataLayer |
|---|---|---|
| `{{pageCategory}}` | Variável da Camada de Dados | `pageCategory` |

### Eventos e Triggers

| Evento | pageCategory | Comportamento |
|---|---|---|
| `view_item_list-DynamicRemarketing` | `Home` | Lê slides ativos do carrossel |
| `view_item_list-DynamicRemarketing` | `InternalSiteSearch` | Lê resultados + MutationObserver para paginação |
| `view_item-DynamicRemarketing` | `Product` | Lê metadados Open Graph do produto |
| `add_to_cart-DynamicRemarketing` | `Product` | Click em `a[href*="cart/add"]` |
| `purchase-DynamicRemarketing` | `purchase` | Busca evento `purchase` GA4 já no dataLayer |

### Trigger GTM sugerido
- **Tipo:** Todas as páginas, **com sequência de tag**
- Configurar para disparar **após** a tag GA4 de purchase (garante que o evento `purchase` já esteja no dataLayer quando o bloco de remarketing de compra for executado)

### Configuração no Google Ads
No Google Ads, certifique-se de que a tag de remarketing está configurada para receber:
- **Tipo de evento:** `view_item`, `view_item_list`, `add_to_cart`, `purchase`
- **Parâmetros:** `value`, `items` (com `id` e `google_business_vertical`)

### Exemplo de dataLayer gerado
```js
// Sempre limpa o campo items antes de cada push (boa prática GA4)
dataLayer.push({ items: null });

dataLayer.push({
  event: 'view_item-DynamicRemarketing',
  value: 299.90,
  items: [{
    id: '7890_SKU001',
    google_business_vertical: 'retail'
  }]
});
```

---

## 🗂️ Estrutura do repositório

```
gtm-ecommerce-snippets/
├── README.md
├── irroba.html          # GA4 DataLayer — Irroba
├── linx.html            # GA4 DataLayer — Linx Commerce
└── vtexdynamicrmkt.html # Dynamic Remarketing — VTEX Legacy
```

---

## 🔧 Compatibilidade

| Snippet | Plataforma de e-commerce | Versão GTM | Padrão de eventos |
|---|---|---|---|
| Irroba | Irroba | v2+ | GA4 |
| Linx | Linx Commerce (EzGaCfg) | v2+ | GA4 |
| VTEX Dynamic Remarketing | VTEX Legacy | v2+ | Google Ads Remarketing |

---

## ⚠️ Notas importantes

- Todos os snippets usam `var` para máxima compatibilidade com ambientes legados
- O padrão `dataLayer.push({ items: null })` antes de cada evento é intencional — evita contaminação de dados entre eventos no GA4
- Teste sempre em ambiente de **Preview** do GTM antes de publicar
- Use o [GA4 DebugView](https://support.google.com/analytics/answer/7201382) e a extensão [GTM/GA Debugger](https://chromewebstore.google.com/detail/gtmga-debugger/ilnpmccnfdjdjjikgkefkcegefikecdc) para validar os eventos

---

## 📬 Contato

Contribuições e issues são bem-vindos! Abra uma issue ou envie um pull request.

---

## 📝 Licença

MIT © — Fique à vontade para usar, modificar e distribuir.
