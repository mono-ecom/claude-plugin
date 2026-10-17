# monobank marketplaces — плагін для Claude

Плагін підключає до Claude маркетплейси monobank:

- **Маркет** — пошук і порівняння товарів, кошик, перехід до оплати в застосунку monobank, історія замовлень і ТТН.
- **Базар** — для ФОП: оголошення, торги, аукціони, замовлення до відправки, виплати; зміна ціни, назви й опису;
  фото; створення й видалення оголошень.

У плагіні є:

| Що | Де | Навіщо |
|---|---|---|
| Конектор `monobank` | `plugins/monobank-marketplaces/.mcp.json` | remote MCP `https://ai.monoecom.tech/mcp`, вхід через OAuth (телефон + push у застосунку) |
| Скіл `market` | `plugins/monobank-marketplaces/skills/market/SKILL.md` | як купувати: від пошуку до посилання на оплату |
| Скіл `bazar` | `plugins/monobank-marketplaces/skills/bazar/SKILL.md` | як вести оголошення, завантажувати фото, створювати нові |

Сервер — [`ecom/services/ai/public-mcp`](https://git.kbyte.app/ecom/services/ai/public-mcp). Опис кожного тулу
живе там, у самих тулах; скіли описують лише сценарії поверх них і не повинні дублювати контракт.

## Встановлення

### claude.ai / Claude Desktop

1. **Customize → Plugins → Add → Add marketplace**, вказати URL цього репозиторію
   (або **Upload plugin** — zip теки `plugins/monobank-marketplaces`).
2. Встановити плагін **monobank-marketplaces**.
3. На вкладці **Connectors** плагіна натиснути **Connect** біля `monobank`: номер телефону → push із кодом у
   застосунку monobank → ввести код.

На Team/Enterprise власник організації може додати маркетплейс в **Organization settings → Plugins** і зробити
плагін «Installed by default», а конектор — в **Organization settings → Connectors**; кожен учасник однаково
логіниться своїм номером.

### Claude Code

```bash
claude plugin marketplace add git@git.kbyte.app:ecom/services/ai/public-mcp-claude-plugin.git
```

```bash
claude plugin install monobank-marketplaces@monobank-ecom
```

Далі `/mcp` → `plugin:monobank-marketplaces:monobank` → Authenticate.

## Фото на Базар: мережевий доступ

Фото не йде через конектор: модель сама виконує `curl` на `https://ai.monoecom.tech/photos/…` зі свого sandbox.
Виклики MCP мережевий доступ не обмежує, а `curl` із sandbox обмежує. **Плагін не може дозволити домен** — у маніфесті
немає такого поля, це налаштування акаунта:

- особистий акаунт: **Settings → Capabilities** → мережевий доступ для code execution → додати `ai.monoecom.tech`;
- Team/Enterprise: власник у **Organization settings → Capabilities** («Package managers + specific domains»);
- Claude Code: `sandbox.network.allowedDomains` у user або project settings.

Без цього завантаження фото й створення оголошень не працюють, а решта тулів працює. Скіл `bazar` розпізнає цю помилку
(status `000`, «CONNECT tunnel failed») і пояснює користувачу, що увімкнути.

## Середовища

`.mcp.json` вказує на prod. Для перевірки на dev/stg замініть URL локально (не комітьте):

| env | URL |
|---|---|
| dev | `https://ai.dev.monoecom.tech/mcp` |
| stg | `https://ai.stg.monoecom.tech/mcp` |
| prod | `https://ai.monoecom.tech/mcp` |

## Зміни

- Новий тул на сервері → додати його в сценарій відповідного скіла, якщо він змінює workflow.
- Перейменування тулу на сервері ламає скіл — шукайте назви `market__*` / `bazar__*` у `skills/`.
- Підніміть `version` у `plugin.json`, щоб клієнти підтягнули оновлення.

```bash
claude plugin validate .
```
