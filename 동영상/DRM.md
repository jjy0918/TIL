# DRM

> https://ottverse.com/what-is-drm-digital-rights-management/

## DRM 이란?

- DRM: Digital Rights Management
- DRM이란 비디오 등 디지털 콘텐츠의 권한을 관리하기 위해 콘텐츠를 암호화하여 전달하고, 복호화하여 소비지가 제어할 수 있도록 도와주는 시스템이다.
- DRM은 콘텐츠를 보호하는 동시에 비즈니스 규칙과 안전한 프로토콜을 통해 콘텐츠를 소비할 사람을 유연한게 선택할 수 있는 방법이다.
- DRM의 간단 프로세스
  - A가 B에게 콘텐츠를 제공한다고 가정하자.
  - A는 B에게 콘텐츠를 바로 전달할 수 없기 때문에 여러 네트워크를 거쳐야 한다.
    - 이때, 콘텐츠는 중간에 누군가에 의해 탈취당할 수 있다.
  - 그래서 A는 콘텐츠를 암호화 하여 키와 함께 B에게 전달한다.
    - 콘텐츠만 탈취 당하면 괜찮지만 키가 같이 탈취 당하면 위험하다.
  - 그래서 A는 제 3자(C)에게 키와 관련된 부분을 위임한다.
    - A는 콘텐츠를 암호화한다.
    - A는 암호화한 키에 대한 정보를 C에게 위임한다.
    - B는 콘텐츠를 복호화 하기 위해 C에게 키에 대한 정보를 요청한다.
    - B가 인가된 사용자면 C는 콘텐츠에 대한 복호화 키를 전달한다.
- 주의할 점은 `DRM`과 `암호화`는 동일하지 않다는 것이다.
  - DRM은 `암호화`를 통해 콘텐츠를 보호한다.
  - 특별한 기술을 사용하여 암호화/복호화 키를 안전하게 저장 및 전달하여 콘텐츠를 보호한다.
  - 콘텐츠를 제공자는 비즈니스 로직을 통해 콘텐츠 소비할 수 있는 사람을 제어할 수 있다.
- 상용 DRM 솔루션
  - Microsoft: PlayReady
  - Google: Widevine
  - Apple: FairPlay

![images](https://ottverse.com/wp-content/uploads/2020/10/with-drm.png)

## EMC, CDM, AES, CENC

> https://ottverse.com/eme-cenc-cdm-aes-keys-drm-digital-rights-management/

![images](https://ottverse.com/wp-content/uploads/2020/10/step-0.png)

1. DRM 서버에 암호화할 키를 요청한다.
2. 받은 키를 이용하여 암호화를 진행한다.
3. 암호화된 컨텐츠를 사용자에 전달한다.
4. 소비자는 DRM 서버에 키를 요청하여 키를 받고 복호화 한다.
5. 소비자는 콘텐츠를 소비할 수 있게 된다.

### ABR(Adaptive BitRate)

- ABR에서는 같은 영상을 다른 비트레이트와 해상도 조합으로 인코딩된 청크나 세그먼트로 분할한다.
- `패키징`이란 영상을 작은 조각으로 쪼개거나 분할하여 매니페스트로 만다는 것을 의미한다.
- ABR에서 사용하는 프로토콜으로는 `MPEG DASH`와 `HLS`가 있다.

### 1단계 비디오 암호화

- 암호화에서 가장 많이 사용되는 기술 중 하나는 `AES(Advanced Encryption Standard)`이다.
  - AES는 대칭키 알고리즘이다.
  - 키의 길이에 따라 128, 192, 256 비트가 있다.(키가 길수록 안전하다.)
  - 키 없이 AES-128을 크래킹하기 위해서는 10억년의 시간이 걸린다고 한다.
- DRM에서 암호화
  - FairPlay: AES-CBC(cbcs)
  - HLS: AES-CBC(cbcs)
  - Widevine, PlayReady: AES-128 CTR(cenc) or AES-128 CBC(cbcs)
  - MPEG-DASH + CMAF: AES-128 CTR(cenc) or AES-128 CBC(cbcs)
  - MPEG-DASH only: AES-128 CTR(cenc)

### 2단계 Key, KeyId, 라이선스 서버

- 암호화 키를 얻는 방법은 수동으로 생성하거나 DRM 공급 업체의 기능을 통해 얻을 수 있다.
- 암호화 키와 암호화된 비디오를 연결하는 것이 KeyId 이다.
- 암호화 Key와 KeyId는 DRM 서버에 저장된다.
- 클라이언트는 KeyId를 가지고 복호화 키를 DRM 라이선스 서버에 요청한다.

![images](https://ottverse.com/wp-content/uploads/2020/10/step1.png)

### 3단계 플레이어와 키 서버에서 비디오 암호화 해독

![images](https://ottverse.com/wp-content/uploads/2020/10/keys-drm.png)

- 위 단계를 통해 클라이언트는 키를 받아와 비디오 복호화를 할 수 있다.
- 여기서 DRM 라이선스 서버는 플레이어가 신뢰할 수 있는지 알 수 없고,
- 플레이어의 복호화 소프트웨어가 키와 복호화된 콘텐츠를 노출할 위험이 있다.
- 또한, DRM 기술에 대한 복호화 모듈개발 필요가 생긴다.

#### CDM(Content Decryption Modules)

- CDM이라는 별도의 암호 해독 모듈을 이용하여 라이선스 요청 생성, 콘텐츠 암호 해득 등 디코딩 처리를 할 수 있다.
- CDM은 Chrome, Firefox, Microsoft Edge, Safari 등과 같은 브라우저에 구축할 수 있다.
- CDM의 소스는 비공개이기 때문에 신뢰할 수 없기는 하다.
- CDM은 암호화를 해독한 후 응용 프로그램이나 디스플레이에 전달한다.

#### EME(Encrypted Media Extentions)

- 브라우저의 CDM은 라이선스 서버와 통신하여야 한다.
- EME는 플레이어가 CDM과 통신할 수 있도록 표준화된 API를 제공한다.
- EME는 JavaScript API이다.
- EME를 사용함에 따라 콘텐츠 제공업체와 플레이어 공급업체가 다른 브라우저에서 스트리밍이 가능해진다.

## Apple FairPlay

> https://ottverse.com/apple-fairplay-drm-how-does-it-work/

### Apple FairPlay란?

- FairPlay는 HLS 프로토콜을 사용하여 스트리밍 하는 컨텐츠를 안전하게 전달하기 위한 Apple의 DRM 솔루션이다.
- FairPlay는 기본적으로 iOS, tvOS, macOS에서 기본적으로 지원이 된다.

### FairPlay Streaming(FPS)의 구성 요소

- HLS 패키징
  - 컨텐츠 암호화 전 HLS 프로토콜을 사용하기 위해 패키징을 해야 한다.
  - HLS는 ABR 스트리밍에 사용된다.
  - 기본적으로 MPEG-TS(ts) 또는 Fragment MP4(fmp4) 컨테이너 형식을 지원한다.
- SAMPLE-AES, AES-128 암호화
  - 패키징 후 AES-128 CBC를 사용하여 암호화를 진행한다.
  - CBC란 Cipher Block Chaining을 의미한다.
    - CBC는 직전 블록 암호화의 출력을 사용하여 현재 블록에 영향을 준다.
    - Initial Vector(IV)는 동일한 입력이 동일한 키로 여러 번 암호화 되더라도 다른 암호문이 생성되도록 사용한다.
    - IV는 랜더마이저처럼 작동하여 해커가 암호문 패턴을 관찰하여 키를 식별하는 것을 방지한다.
- SAMPE-AES
  - 오디오 패킷, 비디오 프레임의 샘플만 CBC와 함께 AES-128을 사용하여 암호화 하는 것.
  - 전체 세그먼트를 암호화할 필요가 없기 때문에 효율적이다.
- AES-128
  - 세그먼트를 128비트의 키와 CBC, PKCS7 패딩으로 완전한 암호화를 진행하는 것.
  - CBC는 IV또는 media-sequnce 번를 사용하여 각 세그먼트 경계에서 다시 시작된다.

### Client Application

- Client Application은 복호화 키를 얻기 위해 License Server에 요청 메시지를 보내는 역할을 한다.
- 콘텐츠 제공자는 자체 FPS 클라이언트 애플리케이션을 개발하거나 DRM 솔루션의 FPS SDK를 사용할 수 있다.

### FPS 동작 과정

![images](https://ottverse.com/wp-content/uploads/2020/10/fairplay-block-diagram-1024x758.png)

1. 사용자가 `재생`을 누른다.
2. Application 재생해야 함을 AVFoundation에 알리고 m3u8 재생 목록에 대한 세부 정보를 제공한다.
3. AVFoundation은 m3u8 파일을 다운로드 받아 구문 분석 한다.
4. AVFoundation은 m3u8에서 #EXT-X-KEY 태그를 찾아 암호화 여부를 확인한다. 암호화가 되어 있다면 App Delegate에 복호화 키를 요청한다.
5. App Delegate는 AVFoundation에게 SPC(Server Playback Context) 메시지 생성을 요청한다.
6. AVFoundation에서 SPC 수신 한 App Delegate는 Key Server로 요청을 보낸다.
   - Key Server의 KSM은 SPC를 언래핑한다.
   - Key Serversms SPC의 정보를 사용하여 콘텐츠 키 조회를 수행한다.
   - 콘텐츠 키는 CKC(Context Key Context)로 래핑하는 FSM으로 전송된다.
7. KSM은 CKC를 App Delegate를 통해 AVFoundation으로 보낸다.
8. AVFoundation은 CKC 내부의 콘텐츠 키를 사용하여 콘텐츠를 복호화하여 사용자에게 보여준다.

## Widevine

> https://ottverse.com/widevine-drm-how-does-it-work/

- Widevine은 Google이 소유한 DRM 솔루션이다.

### Widevine 동작 과정

![images](https://ottverse.com/wp-content/uploads/2020/10/widevine-security-model-1024x768.png)

![image](https://developers.google.com/widevine/drm/overview/fig2.svg)

1. 사용자가 `재생`을 누르면 Application은 `mpd`를 다운받아 Widevine을 사용했는지 판단한 후 initData를 추출하여 플레이어에게 보낸다.
2. 플레이어는 initData를 CDM(Content Decryption Module)로 보낸다.
3. CDM은 플레이어로부터 initData를 받아 라이센스 요청을 생성하여 다시 플레이어에게 보낸다.
4. 플레이어는 라이센스 요청을 받은 후 Widevine 라이센스 서버로 보낸다.
5. 라이센스 서버는 플레이어의 요청에 따라 인증 과정을 거친 후 키와 라이선스에 대한 정보를 포함하여 전달한다.
6. 플레이어는 라이선스 정보를 CDM에게 전달한다.
7. CDM은 OEMCrypto 모듈에게 정보를 전달하여 복호화 및 디코딩을 진행한다.

### Widevine 보안 수준

- L1
  - Widevine에서 가장 높은 수준의 보안.
  - 하드웨어 수준의 암호 해독을 제공한다.
  - 콘텐츠 암호 해독, 미디어 디코딩 및 렌더링 모두 TEE(Trusted Execution Environment) 내에서 수행된다.
- L2
  - L2에서는 미디어 복호화만 TEE 내에서만 수행된다.
  - 디코딩 및 렌더링을 위해서는 애플리케이션으로 전송된다.
- L3
  - L3에서는 암호 해독을 소프트웨어 CDM에서 수행한다.
  - 소프트웨어 암호화

![images](https://ottverse.com/wp-content/uploads/2020/10/widevine-security-levels-1024x288.png)
