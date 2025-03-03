# VOXSDK
- [Возможности](#возможности)
- [Установка](#установка)
- [Использование](#использование)
- [Лицензия](#лицензия)
---

## Возможности

- [x] Создание/отображение статичных баннеров
- [x] Создание/отображение баннеров на полный экран
- [x] Создание/отображение VAST видео-баннеров
- [x] Настройка скрытия баннера
- [x] Настройка времени отображения скрывающей баннер кнопки

## Установка
###### Требования
- iOS 15.0+
- Xcode 14+
- Swift 5.7+
#### Swift Package Manager
1. В Xcode: File > Swift Packages > Add Package Dependency
2. Вставьте URL репозитория: `ssh://git@192.168.1.70:2222/Smorodina/vox-sdk-ios.git`
3. Выберите версию: `Up to Next Major` с версией `1.1.5`

#### CocoaPods
Добавьте в Podfile:
```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '15.0'
use_frameworks!

target 'MyApp' do
  pod 'VOXSDK', :git => "http://192.168.1.70:3000/Smorodina/vox-sdk-ios.git", :branch => "main"
end
```
#### Ручная установка

###### Если вы не используете менеджеры зависимостей, выполните следующие шаги:

1. Инициализируйте git-репозиторий в корне проекта (если его нет):

   ```bash
   $ git init
   ```

2. Добавьте VOXSDK как [подмодуль](https://git-scm.com/docs/git-submodule):

   ```bash
   $ git submodule add http://192.168.1.70:3000/Smorodina/vox-sdk-ios.git
   ```

3. Перетащите VOXSDK.xcodeproj из папки VOXSDK в навигатор вашего проекта Xcode.
4. Убедитесь, что Deployment Target совпадает с вашим приложением.
5. В настройках вашего таргета:
    * Перейдите в General > Embedded Binaries
    * Добавьте VOXSDK.framework (выберите версию для iOS)

## Использование
#### Импортирование
``` Swift
import VOXSDK
```

#### Баннер
###### Метод создания баннера (возвращает представление UIView - сам баннер):

---
``` Swift
VOXAds.createBanner(with: bannerModel)
```
###### Пример использования:

---
``` Swift
Task { @MainActor in
    do {
        let requestModel = AdRequestModel(placeId: "...")
        let bannerModel = BannerModel(
            requestModel: requestModel,
            width: stackView.bounds.width,
            shouldHideBannerWithText: false,
            closeButtonDelay: 5
        )
        let bannerView = try await VOXAds.createBanner(with: bannerModel)
        // Логика размещения баннера
        stackView.addArrangedSubview(bannerView)
    } catch {
        // Обработка ошибки
        print("Ошибка: \(error)")
    }
}
```
###### Параметры BannerModel для настройки баннера

---
``` Swift
/// Модель для запроса рекламного контента
public var requestModel: AdRequestModel
/// Доступная ширина представления
public var width: CGFloat
/// Скрыть баннер с надписью "Реклама скрыта"
public var shouldHideBannerWithText: Bool // Значение по умолчанию 'true'
/// Задержка перед отображением кнопки закрытия
public var closeButtonDelay: TimeInterval // Значение по умолчанию '0.0'
```
###### Размещение баннера

---
* __Ширина__ - задается в `BannerModel.width`
* __Высота__ - рассчитывается автоматически

✅ __Правильный пример:__
``` Swift
bannerView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(bannerView)
NSLayoutConstraint.activate([
    bannerView.topAnchor.constraint(equalTo: view.topAnchor),
    bannerView.centerXAnchor.constraint(equalTo: view.centerXAnchor)
])
```

❌  __Неправильный пример (не фиксируйте высоту!):__
``` Swift
bannerView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(bannerView)
NSLayoutConstraint.activate([
    bannerView.topAnchor.constraint(equalTo: view.topAnchor),
    bannerView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
    bannerView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    bannerView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
])
```
При размещении в `UIStackView` обязательный параметр при вертикальном размещении:
``` Swift
stackView.alignment = .center
```

#### Баннер на весь экран
###### Метод отображения баннера полного экрана:

---
``` Swift
VOXAds.showFullScreenBanner(with: fullScreenBannerModel)
```

###### Пример использования:

---
``` Swift 
Task { @MainActor in
    do {
        let requestModel = AdRequestModel(placeId: "...")
        let fullScreenBannerModel = FullScreenBannerModel(
            requestModel: requestModel,
            presentingViewController: self
        )
        // Экран с баннером отобразится при успехе
        try await VOXAds.showFullScreenBanner(with: fullScreenBannerModel)
    } catch {
        // Обработка ошибки
        print("Ошибка: \(error)")
    }
}
```

###### Параметры FullScreenBannerModel для настройки баннера

---
``` Swift
/// Модель для запроса рекламного контента
public var requestModel: AdRequestModel
/// Контроллер для представления
public var presentingViewController: UIViewController
/// Задержка перед отображением кнопки закрытия
public var closeButtonDelay: TimeInterval // Значение по умолчанию '0.0'
```

#### VAST Видео-баннер
###### Метод отображения видео баннера:

---
``` Swift
VOXAds.createVASTVideo(with: model)
```

###### Пример использования:

---
``` Swift 

let videoBannerView = UIView()
videoBannerView.translatesAutoresizingMaskIntoConstraints = false
addSubview(videoBannerView)
NSLayoutConstraint.activate([
    videoBannerView.centerXAnchor.constraint(equalTo: centerXAnchor),
    videoBannerView.centerYAnchor.constraint(equalTo: centerYAnchor),
    videoBannerView.widthAnchor.constraint(equalToConstant: bounds.width),
    videoBannerView.heightAnchor.constraint(equalTo: videoBannerView.widthAnchor, multiplier: 9.0 / 16.0)
])

Task { @MainActor in
    do {
        let requestModel = AdRequestModel(placeId: "...")
        let model = VideoBannerModel(
            requestModel: requestModel,
            containerView: videoBannerView,
            parentController: controller
        )
        // Экран с видео баннером отобразится внутри контейнера при успехе
        try await VOXAds.createVASTVideo(with: model)
    } catch {
        // Обработка ошибки
        print("Ошибка VOXSDK: \(error)")
    }
}
```

###### Параметры VideoBannerModel для настройки баннера

---
``` Swift
/// Модель для запроса рекламного контента
public var requestModel: AdRequestModel
/// Контейнер для представления
public var containerView: UIView
/// Родительский контроллер представления
public var parentController: UIViewController
/// Скрыть баннер с надписью "Реклама скрыта"
public var shouldHideBannerWithText: Bool // Значение по умолчанию 'true'
/// Задержка перед отображением кнопки закрытия
public var closeButtonDelay: TimeInterval // Значение по умолчанию '0.0'
```

## Лицензия

VOXSDK распространяется под лицензией MIT. [Полный текст лицензии](https://github.com/ktezik/VOXReadme/blob/main/LICENSE).
