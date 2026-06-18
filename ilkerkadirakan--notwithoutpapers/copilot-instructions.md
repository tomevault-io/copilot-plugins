## notwithoutpapers

> Bu repo, Papers Please temali bir RL ortami gelistirir. Ana hedef, kismi gozlemlenebilirlik altinda sinir memuru karar verme problemini modellemektir.

# AGENTS.md

## Proje Ozeti

Bu repo, Papers Please temali bir RL ortami gelistirir. Ana hedef, kismi gozlemlenebilirlik altinda sinir memuru karar verme problemini modellemektir.

- Ortam: Gymnasium
- Ana algoritma: PPO (Stable-Baselines3)
- Cekirdek dosya: `env/papers_env.py`

## Mimari Ilkeleri

- Her episode bir is gunudur.
- Basvuru sahipleri kuyrukta sirayla islenir.
- Ajanin sinirli inceleme zamani vardir.
- Dokuman alanlari baslangicta gizlidir; inceleme aksiyonlariyla acilir.
- Odul tasarimi dogru karar + bilgi toplama maliyeti dengesini hedefler.

## Action/Observation Sozlesmesi

Action ID'leri:
- `0`: APPROVE
- `1`: DENY
- `2`: INSPECT_COUNTRY_ALLOWED
- `3`: INSPECT_HAS_PERMIT
- `4`: INSPECT_EXPIRY_VALID
- `5`: INSPECT_NAME_MATCH
- `6`: INSPECT_HAS_ID_CARD
- `7`: INSPECT_IS_WORKER
- `8`: INSPECT_HAS_WORK_PASS
- `9`: INSPECT_PURPOSE_MATCH
- `10`: INSPECT_SEAL_VALID`r`n- `11`: INSPECT_BIOMETRIC_MATCH

Gozlem vektoru:
- gunluk kural durumu (allowed countries + permit/id card/work pass requirement)
- mevcut basvuru sahibi ulkesi (one-hot)
- alan reveal durumu (unknown/true/false)
- normalize `time_left`
- normalize kalan basvuru sayisi

Bu sozlesmeyi bozan degisikliklerde egitim kodu da ayni committe guncellenmelidir.

## Kod Degisikligi Kurallari

- Deterministik davranisi koru: ayni seed ile ayni episode dagilimi uretilmeli.
- Reward shaping degisikliklerinde eski/yeni metrik farkini acikla.
- Yeni fraud tipi eklenirse:
  - oracle legalite mantigina etkisini acikca yansit
  - observation/action tarafinda hangi alanlarin acildigini netlestir
- Action ID sirasini degistirme. Gerekirse yeni action ekle, mevcutlari kaydirma.
- Gecici `print` debug satirlari birakma.

## Degisim Sonrasi Kontrol Listesi

Kod degisikliginden sonra en az su kontroller yapilmali:

1. Ortam import ediliyor mu:
   - `python -c "from env import PapersPleaseEnv; print('ok')"`
2. Reset/step dongusu calisiyor mu:
   - Rastgele policy ile en az 1 episode don.
3. Observation boyutu `observation_space` ile tutarli mi.
4. `terminated` / `truncated` bayraklari beklenen sekilde set ediliyor mu.
5. `episode_stats` alanlari korunuyor mu.

## Degerlendirme Metrikleri

Asagidaki metrikler raporlanmalidir:

- decision accuracy
- false accept rate
- false reject rate
- inspection frequency
- episode reward

## Gelecek Genisletmeler

Planlanan genisletmeler:

- ek dokuman turleri
- daha fazla fraud paterni
- ulke/rule cesitliligi
- episode icinde dinamik kural degisimi
- hafif bir gorsellestirme katmani (Tkinter / PyGame / Streamlit)

Genisletmelerde once minimal ortam kararliligi korunmali, sonra complexity artirilmalidir.

---
> Source: [ilkerkadirakan/NotWithoutPapers](https://github.com/ilkerkadirakan/NotWithoutPapers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
