# Programming iOS 14 - I. Views

## Views

뷰란 무엇인가
* 뷰는 사각형의 인터페이스 공간에 자기 자신을 어떻게 그려야할지 알고 있는 객체다
* 어플리케이션은 뷰를 조작해 사용자에게 시각적 효과를 줄 수 있다
* 뷰를 조작하는 가장 간단한 방법은 뷰를 설정하고 잊어버리는 것이다
  - nib 에디터로 버튼을 만들고 앱을 실행시키면 버튼이 보일 것이다
  - 참고로 뷰를 nib로 만들 것인가 아니면 코드로 만들 것인가는 어플리케이션의 아키텍쳐나 취향에 달려있다
* 하지만 현실의 애플리케이션은 보다 다채로운 기능을 요구한다

Responder로서의 뷰
* 보다 파워풀한 앱을 만들기 위해서는 특정 뷰를 보이게 하거나, 사라지게 하거나, 이동시키거나 리사이징하는 등의 작업이 필요하다
* 이처럼 뷰는 보여지는 것이기도 하지만 유저로부터 탭이나 스와이프 같은 입력을 받아 반응하는 객체이기도 하다
* 유저의 입력을 적절히 처리함으로써 보다 파워풀한 앱이 된다

View hierarchy and Responder chain
* 뷰 조작에 있어 반드시 알아야 할 것은 뷰가 계층 구조로 이루어져 있다는 것이다
* 하나의 뷰는 복수의 서브뷰들을 가질 수 있다
  - 반대로 서브뷰는 하나의 슈퍼뷰를 가진다
* 이런 구조를 일반적으로 뷰 트리라고 한다
* 뷰 트리 구조에서 슈퍼뷰와 서브뷰는 특정 작업에 함께 영향을 받는다
  - ex1. 슈퍼뷰가 제거되면, 서브뷰도 제거된다
  - ex2. 슈퍼뷰가 이동되면, 서브뷰도 이동된다
* 이처럼 슈퍼뷰에 어떤 반응이 일어나면 서브뷰에도 영향을 미치는 구조를 Responder chain라 한다

---

## Window scene architecture

iOS 13 시작시
* 앱 윈도우의 뒤에는 윈도우 씬이 위치하고 있다
* 이 점이 아키텍쳐에 있어 iOS 12와 비교했을 때의 주된 변화다

윈도우 아키텍쳐
* iOS 12나 그 이전의 아키텍쳐다
* 윈도우 베이스이며, 윈도우는 app delegate의 프로퍼티다
* 만약 Xcode 10이나 그 이전 버전에서 앱을 만들면 이 아키텍쳐를 사용한다

씬 아키텍쳐
* iOS 13이나 그 이후의 아키텍쳐다
* 씬 베이스이며, 윈도우는 scene delegate의 프로퍼티다
* Xcode 11이나 그 이전 버전에서 앱을 만들면 이 새로운 아키텍쳐를 사용한다

iOS 13 이상이지만 윈도우 아키텍쳐로 앱이 구성된 경우
* 윈도우 씬이 주어지지만 그럼에도 앱에는 인식되지 않으므로 영향이 없다
* 여전히 앱은 윈도우 아키텍쳐로 실행된다
* 반대로 새로운 아키텍쳐도 iOS 12 이하에서는 실행될 수 없다

구체적으로 앱에서 어떤 코드가 아키텍쳐를 지정하고 있는지 알아보기
* XCode 12 이상에서 새로운 앱을 만들면 다음과 같은 코드가 생성되어 있을 것이다

Scene delegate
```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?
    // ...
}
```
* 앱은 scene delegate를 가지며 window 프로퍼티를 포함하고 있다

씬 베이스 앱 델리게이트 메소드
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    // ...
    func application(_ application: UIApplication,
        configurationForConnecting connectingSceneSession: UISceneSession,
        options: UIScene.ConnectionOptions) -> UISceneConfiguration {
            // ...
    }
    func application(_ application: UIApplication,
        didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
            // ...
    }
}
```
* app delegate는 씬을 참조하는 두 개의 메소드를 가지고 있다

Application Scene Manifest
* Info.plist 파일을 보면 Application Scene Manifest가 포함되어 있는 것을 확인할 수 있을 것이다

```txt
<key>Application Scene Manifest</key>
<dict>
    <!-- ... -->
</dict>
```

iOS 12나 이전버전에서 새로운 아키텍쳐를 사용하려는 경우
1. window 프로퍼티 선언을 AppDelegate 클래스에 복사 붙여넣기 한다. 그럼 AppDelegate, SceneDelegate는 window 프로퍼티를 가지고 있게 된다
2. AppDelegate 클래스에서 UISceneSession을 참조하는 특정 메소드를 다음 코드로 마킹한다
```swift
@available(iOS 13.0, *)
```
3. SceneDelegate 클래스에서 전체 클래스를 위 코드로 마킹한다

위 코드들의 결과
* iOS 13 이상의 기기에서는 씬 델리게이트의 윈도우 프로퍼티가 윈도우를 가지는 새 아키텍쳐를 사용한다
* 그 이전 버전에서는 앱 델리게이트의 윈도우 프로퍼티가 윈도우를 가지는 이전 아키텍쳐를 사용한다
* 생명주기 관련 메시지와 노티피케이션에서도 차이가 있다
  - iOS 13 이상의 기기는 씬 델리게이트에서 sceneDidBecomeActive(_:)로 메시지를 받는다
  - 그 이전 버전에서는 앱 델리게이트에서 applicationDidBecomeActive(_:)로 메시지를 받는다
* 지금까지의 내용들이 너무 거추장스러워서 iOS 13 이후에도 이전 아키텍쳐를 그대로 사용하고 싶다면 다음과 같이 하면 된다
  - UISceneSession이나 SceneDelegate 관련 코드를 전부 삭제한다
  - Info.plist 파일에 있는 Application Scene Manifest를 지운다

---

## 앱은 어떻게 런치되는가

단일 호출 지점
* 모든 앱은 UIApplicationMain를 통해 시작된다
  - Obj-C로 작성된 앱과 다르게 스위프트에서는 명시적으로 이 함수를 호출하지는 않고 씬 뒤에서 호출된다
* 위 함수의 호출로 앱의 초기 인스턴스들이 생성된다
* 만약 앱이 main 스토리보드를 사용하면, 이 인스턴스들은 윈도우와 루트 뷰 컨트롤러에 포함된다
* 그런데 보다 정확히는 UIApplicationMain는 아키텍쳐(old/new)에 따라 다르게 작동한다

iOS 13 이상 버전에서 UIApplicationMain는의 작동방식
1. UIApplicationMain은 UIApplication을 초기화하고 앱 전체에 공유되는 어플리케이션 인스턴스들을 제공하기 위해 UIApplication에서 생성된 인스턴스들을 소유한다. 이렇게 생성된 공유 인스턴스들은 UIApplication.shared로 참조가능하다. 그리고나서 UIApplicationMain는 앱 델리게이트 클래스를 초기화한다(클래스에 @main이 마킹되어 있다, 스위프트 5.3 이전에는 @UIApplicationMain). 이 클래스는 앱 델리게이트 인스턴스들을 소유하며, 인스턴스들은 앱이 살아있는 동안 유지된다
2. UIApplicationMain은 앱 델리게이트의 application(_:didFinishLaunchingWithOptions:)를 호출한다
3. UIApplicationMain은 UISceneSession, UIWindowScene, 윈도우 씬의 델리게이트로 제공될 인스턴스를 생성한다. Info.plist 파일에 윈도우 씬 델리게이트 인스턴스의 클래스를 문자열로 지정한다(Application Scene Manifest 딕셔너리에 있는 Scene Configuration에 Delegate Class Name을 지정되어 있다). 빌트인 템플릿에서 이 클래스는 SceneDelegate 클래스다. 빌트인 템플릿으로 프로젝트를 생성한 후 Info.plist 파일에 $(PRODUCT_MODULE_NAME)로 쓰여져있다. 
4. UIApplicationMain은 첫 번째 씬이 스토리보드를 사용하고 있는지를 알아본다. Info.plist의 Application Scene Manifest 딕셔너리에 있는 Scene Configuration에 Storyboard Name에 지정되어 있다. 만약 지정되어 있다면, UIApplicationMain은는 스토리보드의 초기 뷰컨트롤러를 초기화한다