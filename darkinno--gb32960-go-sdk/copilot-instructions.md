## gb32960-go-sdk

> GB/T 32960 �萄瘙質膠餈���縑�悅 Go SDK��?T-BOX TCP 餈�圾��霈格�柴�靚��o ��摨����嗅��其�韏��瘀�Go �垢撘�?

# Project: gb32960

## 憿寧�膩

GB/T 32960 �萄瘙質膠餈���縑�悅 Go SDK��?T-BOX TCP 餈�圾��霈格�柴�靚��o ��摨����嗅��其�韏��瘀�Go �垢撘�?
## 蝻���

- Go 隞���萄儐 Effective Go �?Go Code Review Comments
- ��撠��嚗codec`�constant`�auth`�forward`嚗?- 撖澆蝚血�?PascalCase嚗�� camelCase
- �秤��隞?`Err` 撘憭湛�憒?`ErrInvalidStart`嚗?- 撣賊�隞亦掩��蝻嚗Cmd`�Enc`�Field`嚗�蝏?- 雿輻 `go vet` 蝖桐��誨���?- 蝳迫雿輻 `panic` 憭�銝�餉�嚗�?`errors.New` / `fmt.Errorf`

## ��蝥血�

```
Server (�砍� API)
  ��� Connection Manager (TCP 餈蝞∠�)
       ��� Decoder (蝎�/撣扯圾�?
            ��� Message Processor (�賭誘��)
                 ��� Handler (�冽��)
                 ��� Authenticator (霈方�)
                 ��� Forwarder (頧砍�)
```

- Handler �亙摰��?`events.go`嚗��啣�冽憿寧
- �悅蝻圾� `decoder.go` + `codec/`嚗�靘� Server
- ��楊璅∪�靚���亙嚗Authenticator`�Forwarder`嚗?
## 擐極�?摨?
- �詨�蝵�嚗net` ��摨?- �亙�嚗log/slog`嚗o 1.21+嚗?- 撟嗅�嚗sync` + goroutine
- 瘚�嚗testing` ��摨��洵銝�剛�摨�
- ����`go vet`

## 瘚�

- 餈��券瘚�嚗go test ./... -count=1`
- 餈��葵��霂�`go test ./codec/ -v`
- 瘚��辣�賢�嚗*_test.go`嚗�皞�隞嗅��?- 銝蝙�?assert/mock 摨��湔雿輻 `testing.T` �寞�
- 靽格�悅閫���餉���憿餉�銵�冽�霂?
## 蝥行�

- **銝�撘蝚砌��嫣�韏?*�?`go.mod` �寞芋�forward/` 銝剔��拙��� build tag �氖
- **銝��游� Handler �亙**�憓�靚瘜�閬����澆捆
- **銝�靽格 `constant/` �葉�虜�啣?*嚗�?GB32960 ���漣
- **�唳摮挾閫��敹◆�?`codec/` �葉**嚗�閬 `message.go` 銝剖�鋆詨��圾�?- **��?TCP 霂餃��?`conn.go` 銝?*嚗�閬 `server.go` 銝剜�雿?`net.Conn`
- **Server �?`connections` sync.Map 銝��湔��**嚗 `Connections()` �寞�
- **憒�瘛餃��啁��賭誘憭�**嚗 `conn.go` �?`handlePacket` 銝剜溶�?case

## 撣貊�賭誘

| �� | �賭誘 |
|---|---|
| 蝻�璉�?| `go build ./...` |
| 餈�瘚� | `go test ./... -count=1` |
| ���?| `go vet ./...` |
| 蝻�蝷箔� | `go build -o server ./cmd/example/` |
| Kafka 頧砍�蝷箔� | `go build -tags kafka -o server ./cmd/example/` |
| MQTT 頧砍�蝷箔� | `go build -tags mqtt -o server ./cmd/example/` (��溶���辣) |

## �桀�蝏�

```
gb32960/
��� server.go / options.go / events.go / interfaces.go   # �砍� API
��� decoder.go / packet.go / message.go / buffer.go       # �悅撅?��� conn.go / vin_registry.go                             # 餈撅?��� constant/command.go                                   # �悅撣賊�
��� codec/{field,vehicle,motor,battery,engine,position,extreme,alarm,voltage}.go  # �唳蝻圾�?��� auth/auth.go                                          # 霈方�摰
��� forward/config.go                                     # 頧砍��蔭
��� cmd/example/main.go                                   # 蝷箔��亙
��� *_test.go                                             # 瘚��辣
```

---
> Source: [DarkInno/gb32960-go-sdk](https://github.com/DarkInno/gb32960-go-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
