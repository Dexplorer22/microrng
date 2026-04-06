# 🔒 КАК СДЕЛАТЬ РЕПОЗИТОРИЙ ПРИВАТНЫМ

**Репозиторий:** https://github.com/Dexplorer22/microrng

## Способ 1: Через веб-интерфейс GitHub (самый простой)

1. Перейди на https://github.com/Dexplorer22/microrng
2. Нажми на **Settings** (шестерёнка в верхнем меню)
3. Прокрути вниз до **Danger Zone**
4. Нажми на **Change repository visibility**
5. Выбери **Make private**
6. Подтверди действие

## Способ 2: Через GitHub API (если нужно автоматизировать)

```bash
curl -X PATCH \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/Dexplorer22/microrng \
  -d '{"private":true}'
```

## Способ 3: Через GitHub CLI (если установлен)

```bash
gh repo edit Dexplorer22/microrng --private
```

---

## ✅ После того как репозиторий станет приватным:

- ✅ Только ты сможешь видеть репозиторий
- ✅ Только приглашённые люди смогут получить доступ
- ✅ Все документы будут защищены
- ✅ Все commits будут приватными
- ✅ Никто не сможет форкнуть репозиторий

---

**Рекомендуем:** Используй Способ 1 (веб-интерфейс) - это самый простой способ.

