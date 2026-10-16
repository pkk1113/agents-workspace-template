# 전역 데이터

`ConstData`, `RuntimeData`, `LocalData`에 값을 추가·수정하거나, 전역 값·`PlayerPrefs`를 다룰 때 읽는다.

`ConstData`, `RuntimeData`, `LocalData`는 앱 전체의 전역 데이터 경계이며 `Assets/__App__/Data/Scripts`에 둔다. 이 데이터는 Composition만 직접 알고, Feature에는 필요한 값, Property 또는 Command를 Composition이 전달하도록 한다. 목적: Feature가 전역 상태에 숨은 의존을 갖지 않게 해 독립적으로 변경·테스트할 수 있게 한다.

- `ConstData`는 앱 전체에서 변하지 않는 공통 데이터를 둔다. 컴파일 타임 리터럴은 `const`로, `Color`, `Vector3`, 배열, 클래스처럼 `const`가 불가능한 고정 값은 `static readonly`로 둔다.
- `RuntimeData`는 실행 중인 앱 세션의 값과 실행 환경 데이터를 둔다. 앱 시작·종료 시 초기화 책임을 분명히 한다.
- `LocalData`는 `PlayerPrefs`를 감싼 로컬 저장소다. 다른 코드가 `PlayerPrefs` 키를 직접 읽거나 쓰지 않는다.

전역 데이터 안에서 의미 있는 구분이 필요하면 `public static class`를 중첩해 `ConstData.SomePropKeys.AKey`처럼 접근한다. 에셋 종류나 기술적 분류가 아니라 데이터의 의미와 책임을 기준으로 묶는다.

```csharp
public static class ConstData
{
    public const string MODEL_CACHE_PATH = "model_cache";
    public static readonly Color OUTLINE_COLOR = Color.white;
    public static readonly Vector3 MENU_OFFSET = new Vector3(0f, 0.1f, 0f);

    public static class ServerInfoKey
    {
        public const string MQ_IP = "server.mq.ip";
    }
}

public static class RuntimeData
{
    public static string HardwareId { get; set; } = Util.GetHardwareId();
    public static UserProfile CurrentUser { get; set; }
}

public static class LocalData
{
    public static class RemoteServer
    {
        private const string IP_KEY = "remote_server_ip";

        public static string Ip
        {
            get => PlayerPrefs.GetString(IP_KEY);
            set => PlayerPrefs.SetString(IP_KEY, value);
        }
    }

    public static class Connection
    {
        private const string AUTO_LOGIN_KEY = "auto_login";

        public static bool AutoLogin
        {
            get => PlayerPrefs.GetInt(AUTO_LOGIN_KEY, 0) == 1;
            set => PlayerPrefs.SetInt(AUTO_LOGIN_KEY, value ? 1 : 0);
        }
    }
}

LocalData.RemoteServer.Ip = "192.168.0.10";
LocalData.Connection.AutoLogin = true;
```
