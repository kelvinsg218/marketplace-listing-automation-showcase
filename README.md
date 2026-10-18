# MLA — Marketplace Listing Automation

**Do catálogo da empresa ao anúncio revisado, em escala.**
Aplicativo desktop para Windows que organiza a base de produtos de um lojista, prepara anúncios por marketplace e
por conta e automatiza o preenchimento no navegador, sempre com revisão humana antes de publicar.

> Este repositório é a **apresentação visual e comercial** do MLA. O código-fonte, os testes e a documentação
> técnica ficam em um repositório privado.
> Página de apresentação: [`index.html`](index.html) (pronta para o GitHub Pages).

![Dashboard do MLA](docs/images/dashboard.png)

| Situação | |
|---|---|
| Versão | 0.1.0 + etapa “Catálogo Mestre Inteligente” |
| Plataforma | Windows 10/11 (desktop) |
| Testes automatizados | 178 (unidade, interface e ponta a ponta no Microsoft Edge) |
| Publicação real no Mercado Livre e na Shopee | **Em desenvolvimento** — hoje o fluxo completo roda no marketplace simulado incluído |

As capturas abaixo são da interface real do aplicativo, com dados fictícios de demonstração. Os anúncios marcados
como publicados nelas foram publicados no **marketplace simulado**, não em um marketplace real.

---

## O problema
Lojistas que vendem em vários marketplaces costumam ter:
- a base de produtos espalhada em planilhas do ERP, XML, PDFs de fornecedores e pastas de fotos;
- SKUs, preços e estoques divergentes entre canais, sem saber de onde veio cada informação;
- compatibilidades (veículos, aparelhos, equipamentos) anotadas sem fonte e misturadas com suposições;
- trabalho repetitivo e arriscado para adaptar e publicar o mesmo produto em cada marketplace e conta.

## A solução
O MLA mantém um **catálogo mestre** único e confiável, importa bases completas com revisão linha a linha, separa o
produto físico dos seus **anúncios** por marketplace e conta e conduz a publicação por uma **fila com revisão
humana obrigatória**.

---

## Funcionalidades implementadas

### Catálogo mestre inteligente
Identificação completa (SKU principal e de variações, código do fabricante, GTIN/EAN com dígito verificador
conferido, marca, modelo, categoria, fornecedor, nome comercial), dados comerciais (custo, preço de venda e
promocional, estoque e estoque mínimo, unidade, garantia, condição, situação comercial), dados técnicos
(descrição original e técnica, material, cor, peso, dimensões, especificações, atributos personalizados) e mídia
(imagens, variações, arquivos técnicos e mídias pendentes). Checagem ao vivo do que falta para anunciar.

| Catálogo | Editor do produto |
|---|---|
| ![Catálogo mestre](docs/images/catalogo-mestre.png) | ![Editor do produto](docs/images/editor-produto.png) |

### Gerenciamento de SKUs, preços, estoque e variações
Busca por SKU (inclusive de variação), GTIN, código do fabricante, marca ou fornecedor; SKUs duplicados
bloqueados; variações com SKU, preço e estoque próprios; filtros por situação, mídia pendente e grupo; listas
grandes carregadas aos poucos.

### Central de Importações
CSV, XLSX, JSON, XML, PDF (tabela ou ficha técnica), ZIP com planilha e fotos e pastas de imagens nomeadas pelo SKU.

| Mapeamento das colunas | Revisão antes de gravar |
|---|---|
| ![Mapeamento](docs/images/importacao-mapeamento.png) | ![Revisão](docs/images/importacao-revisao.png) |

- Reconhecimento automático de colunas; nomes ambíguos (`codigo`, `saldo`) exigem confirmação; modelos salvos.
- Validação por linha, duplicatas, variações ligadas ao SKU pai, associação de imagens por nome ou SKU.
- Cada alteração aparece como “valor atual → valor do arquivo”; o usuário decide criar, atualizar ou ignorar.
- Nada é gravado antes da confirmação; gravação de uma vez só (cancelar não altera nada); relatório por linha.
- Arquivos executáveis, scripts e planilhas com macro dentro de ZIPs são ignorados e nunca executados.

### Compatibilidade de produtos
Marca, modelo, versão, anos, equipamento e código de referência, com fonte e estado de verificação. Dados do
cliente entram como confirmados; fabricante e fontes externas, como não verificados. Nada é inferido.

![Compatibilidades](docs/images/compatibilidades.png)

### Preparação de anúncios por marketplace e conta
Um produto mestre gera um anúncio por marketplace e conta, guardando só o que muda (título, descrição, preço,
estoque, categoria, atributos, imagens). Estados: Rascunho, Incompleto, Pronto para revisão, Aprovado, Na fila,
Em execução, Publicado confirmado, Falha e Requer intervenção. Nota de completude calculada sem IA.

![Preparação de anúncios](docs/images/preparacao-anuncios.png)

### Marketplaces e contas
Várias contas por marketplace, conta padrão, URL do vendedor validada pelo domínio oficial, navegador Edge
dedicado e isolado por conta e teste de conexão com login manual.

![Marketplaces e contas](docs/images/marketplaces-contas.png)

### Automação, filas e grupos
Fila persistente (um item por vez), pausa, retomada, cancelamento, recuperação após fechar o app, prevenção de
duplicados, capturas de tela de cada etapa e **aprovação humana antes de publicar**. Execução individual, em grupo
ou em lote.

| Automações | Grupos |
|---|---|
| ![Automações](docs/images/automacoes.png) | ![Grupos](docs/images/grupos.png) |

### Dashboard e histórico de operações
Pipeline do catálogo e dos anúncios, pendências, situação por marketplace e histórico com filtros, detalhes,
reexecução segura e exportação CSV.

![Histórico](docs/images/historico.png)

### Marketplace simulado
Marketplace local com login, CAPTCHA de teste, formulário e confirmação, para demonstrar o fluxo completo sem
publicar nada de verdade.

---

## Arquitetura
```
Interface (PySide6)  ──▶  Serviços (regras de negócio)  ──▶  Persistência (SQLite, migrações versionadas)
        │                         ▲
        │                         │
        └──▶ Importação (CSV/XLSX/JSON/XML/PDF/ZIP → plano → revisão → gravação em transação)
        └──▶ Automação (thread de execução, Playwright) ──▶ Adaptadores por marketplace ──▶ Microsoft Edge
```
- Dados locais em `%LOCALAPPDATA%\MLA`, backup automático antes de cada atualização do banco.
- Produto mestre separado do anúncio; cada anúncio guarda só as diferenças em relação ao produto.
- Dados do cliente, informações externas e sugestões de IA ficam em estruturas separadas.

## Tecnologias
Python 3.13 · PySide6 (Qt) · SQLite (WAL) · Playwright + Microsoft Edge · openpyxl · pypdf · pytest

## Princípios
- Nada é publicado sem aprovação; “publicado” só com confirmação do marketplace.
- Login, CAPTCHA e verificações são sempre feitos pelo usuário; nenhum bloqueio é contornado.
- Importações nunca apagam registros nem substituem dados sem mostrar a alteração.
- Hipóteses nunca são apresentadas como fatos.

---

## Em desenvolvimento
- **Publicação real no Mercado Livre** — depende da inspeção autorizada do formulário real da conta do vendedor.
- **Publicação real na Shopee** — depois do Mercado Livre.
- Estrutura já preparada (sem telas e sem coleta ativa): URLs de referência de concorrentes com fonte e data,
  contrato para coletores (API oficial primeiro; Scrapling só quando permitido) e armazenamento de sugestões de IA
  aplicadas apenas com aceite do usuário.

## Roadmap
| Etapa | Situação |
|---|---|
| Base do app, catálogo, grupos, fila com revisão, marketplace simulado | Concluído |
| Catálogo mestre, Central de Importações, compatibilidades, preparação de anúncios, contas | Concluído |
| Inspeção autorizada e integração real com o Mercado Livre | Próxima etapa |
| Integração real com a Shopee | Planejado |
| Telas de regras por categoria, consentimento de IA e imagens por variação | Planejado |
| Instalador, licenças assinadas e teste em máquina limpa | Planejado |
| Inteligência de mercado (APIs oficiais) e agentes de otimização com IA | Planejado |

---

O código-fonte, os testes e a documentação técnica do MLA são mantidos em um repositório privado.
