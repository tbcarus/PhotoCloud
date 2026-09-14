# Профиль и настройки

## 1. Сведения о текущем входе

**Status:** DEFERRED  
**Availability:** ANDROID_ONLY  
**System:** [PROFILE-AUTH](../system/03-profile-and-settings.md#profile-auth)

После входа Android показывает email текущей локальной сессии и даёт возможность проверить доступ или выйти. Это краткая информация о текущем входе, а не полноценный профиль пользователя.

## 2. Просмотр профиля

**Status:** DEFERRED  
**Availability:** SERVER_ONLY  
**System:** [PROFILE-VIEW](../system/03-profile-and-settings.md#profile-view)

Server умеет вернуть сведения профиля текущего пользователя. Android эти сведения сейчас не запрашивает и не показывает отдельным рабочим экраном профиля.

## 3. Изменение профиля

**Status:** DEFERRED 
**Availability:** STUB  
**System:** [PROFILE-EDIT](../system/03-profile-and-settings.md#profile-edit)

Редактирование профиля сейчас не работает как функция продукта: экран Profile в Android является заглушкой, а изменение профиля на Server не доведено до рабочей операции.

## 4. Пользовательские настройки

**Status:** DEFERRED 
**Availability:** STUB  
**System:** [SETTINGS-USER](../system/03-profile-and-settings.md#settings-user)

Экран Settings сейчас является заглушкой. Пользователь не может через него управлять автоматическим backup, выбирать режим только по Wi-Fi, менять интервал, правила повторов, набор папок или медиа и другие параметры поведения синхронизации.

## 5. Выбор Server и проверка соединения

**Status:** DEFERRED  
**Availability:** END_TO_END  
**System:** [SETTINGS-NETWORK](../system/03-profile-and-settings.md#settings-network) · [SYSTEM-SCOPE](../system/07-system-rules-and-recovery.md#system-scope)

На отдельном экране Network пользователь вручную указывает адрес Server и проверяет соединение. После успешной проверки этот Server становится текущим для дальнейших запросов приложения.

Смена Server сама по себе не означает смену аккаунта и не очищает предыдущую локальную историю. Поэтому новый Server может сочетаться с уже сохранённой сессией и очередью от прежнего контекста.

## 6. Смена email, avatar и удаление аккаунта

**Status:** DEFERRED  
**Availability:** NOT_PRESENT  
**System:** [PROFILE-ABSENT](../system/03-profile-and-settings.md#profile-absent)

Смена email, avatar и удаление аккаунта как пользовательские функции PhotoCloud сейчас отсутствуют.
