# 🤖 WebWorld BetBot Beta Prototype  
### 🎯 Автоматизированный веб-агент для анализа и выполнения букмекерских ставок (на основе ATP-тенниса)

[![License: GPL-3.0](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-red?logo=pytorch)](https://pytorch.org/)
[![Hugging Face](https://img.shields.io/badge/HF_Transformers-4.40%2B-green?logo=huggingface)](https://huggingface.co/)
[![Selenium](https://img.shields.io/badge/Selenium-4.0%2B-orange?logo=selenium)](https://www.selenium.dev/)

---

## 📋 Содержание

- [Концепция](#concept)
- [Архитектура](#architecture)
- [План реализации](#plan)
- [Возможности](#capabilities)
- [Ограничения](#limitations)
- [Требования](#requirements)
- [Лицензия](#license)

## 🎯 Концепция <a name="concept"></a>

Beta-прототип демонстрирует гибридную архитектуру:
- **Модель мира для веба** (дообученная на русском языке) управляет *навигацией и взаимодействием* с сайтом букмекера (`fon.bet`, `1xbet`, и др.).
- **Внешний аналитический ИИ** (`qwen-coder` или аналог) получает данные из интернета, анализирует статистику матчей ATP и принимает решение о ставке.
- **Selenium** выполняет физические действия в браузере по командам модели.
- **`send_msg_to_user`** используется как *интерфейс запроса* к внешней модели (анализ → решение → возврат в цикл).

> Цель: не "предсказание", а **автоматизация процесса ставки** на основе внешнего анализа.

## 🏗️ Архитектура <a name="architecture"></a>

- **WebWorld** — *не прогнозирует исход*, а *планирует и исполняет веб-действия*.
- **qwen-coder** — *думает*: получает статистику, сравнивает игроков, выдаёт рекомендацию («ставка на A, сумма 100»).
- **Python Agent** — координатор: захват состояния → вызов WebWorld → обработка `send_msg_to_user` → вызов qwen-coder → возврат ответа → выполнение через Selenium.

## 🚀 План реализации <a name="plan"></a>

| Этап | Действие |
|------|----------|
| 1️⃣ | Установка зависимостей: `torch`, `transformers`, `selenium`, `webdriver-manager`, `playwright` (для A11y Tree) |
| 2️⃣ | Загрузка `WebWorld-8B` и `WebWorldData` через `huggingface_hub` |
| 3️⃣ | Перевод датасета на русский (с `qwen-coder`) + ручная проверка |
| 4️⃣ | Дообучение (`LoRA`/`QLoRA`) на русскоязычном датасете + ваших собственных траекторий (например, `fon.bet` + `send_msg_to_user("ATP: [A] vs [B]")`) |
| 5️⃣ | Реализация агента: цикл `state → WebWorld → action / send_msg → qwen-coder → state' → Selenium` |
| 6️⃣ | Тестирование на реальных матчах ATP (с имитацией ставок без денег) |

## ✅ Возможности <a name="capabilities"></a>

- 🌐 Работа с русскоязычными сайтами (благодаря дообучению).
- 🧠 Интеграция внешнего анализа через `send_msg_to_user`.
- ⚙️ Поддержка длинных цепочек действий (до 30+ шагов).
- 📦 Готовый пайплайн: от анализа до клика «Сделать ставку».

## ⚠️ Ограничения <a name="limitations"></a>

- ❌ **WebWorld не предсказывает исходы** — она только *выполняет действия* по вашему плану.
- ❌ Точность ставок зависит *только* от качества внешней модели (`qwen-coder`), а не от WebWorld.
- ⚖️ Букмекеры блокируют ботов — требуется обход (user-agent, прокси, задержки).
- 📜 GPL-3.0: весь исходный код должен быть открыт и распространён под той же лицензией.

## 🛠️ Требования <a name="requirements"></a>

| Компонент | Версия | Примечание |
|-----------|--------|------------|
| Python    | ≥ 3.8  | — |
| `torch`   | ≥ 2.0  | Для запуска моделей |
| `transformers` | ≥ 4.40 | Hugging Face API |
| `huggingface_hub` | ≥ 0.20 | Загрузка моделей |
| `selenium` | ≥ 4.0 | Автоматизация браузера |
| `webdriver-manager` | ≥ 4.0 | Автоустановка ChromeDriver |
| `playwright` | ≥ 1.30 | Генерация A11y Tree (опционально, но рекомендуется) |
| GPU       | ≥ 24GB VRAM (для 8B) | Для fine-tuning; inference — можно и на 16GB |

> 💡 Для тестирования можно начать с `WebWorld-8B` в режиме `torch.bfloat16` и `device_map="auto"`.

## 📄 Лицензия <a name="license"></a>

Этот проект распространяется под лицензией **GNU General Public License v3.0** ([текст лицензии](https://www.gnu.org/licenses/gpl-3.0.html)).  
Все производные работы должны быть опубликованы с открытым исходным кодом под той же лицензией.

> ⚠️ Исходные модели (`WebWorld`, `Qwen3`) имеют свою лицензию (Apache 2.0), но *ваш код* — под GPL-3.0.
