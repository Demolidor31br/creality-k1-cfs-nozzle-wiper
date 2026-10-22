# 🧹 Creality K1 / K1C / K1 Max - CFS Safe Nozzle Wiper

[![Klipper](https://img.shields.io/badge/Klipper-Compatible-orange.svg)](https://www.klipper3d.org/)
[![OrcaSlicer](https://img.shields.io/badge/OrcaSlicer-Calibrated-blue.svg)](https://github.com/SoftFever/OrcaSlicer)
[![Creality K1](https://img.shields.io/badge/Creality-K1%20%7C%20K1C%20%7C%20K1%20Max-green.svg)](https://www.creality.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![YouTube Shorts](https://img.shields.io/badge/YouTube-Assista%20o%20V%C3%ADdeo-red?logo=youtube)](https://youtube.com/shorts/4Iyi-nFR4YI)

Rotina calibrada de **limpeza mecânica de bico com escova de silicone** para impressoras **Creality K1, K1C e K1 Max** equipadas com o sistema multi-cor **CFS (Creality Filament System)**.

Desenvolvido para eliminar o gotejamento de plástico pós-purga, evitar contaminação de cores e prevenir colisões da mesa contra o mecanismo da escova.

---

## 📺 Demonstração em Vídeo (YouTube Shorts)

Assista ao ciclo completo em funcionamento na máquina:  
▶️ **[Clique aqui para assistir no YouTube Shorts](https://youtube.com/shorts/4Iyi-nFR4YI)**

<p align="center">
  <img src="images/capa_youtube_shorts.jpg" width="380" alt="Demonstração do Nozzle Wiper na K1" />
  &nbsp;&nbsp;&nbsp;&nbsp;
  <img src="images/nozzle_wiper_macro.jpg" width="450" alt="Bico entrando na escova de silicone" />
</p>

---

## 🎯 Problemas Resolvidos por Este Mod

1. **Gotejamento & Fiapos Pós-Purga:** Após a purga no bueiro traseiro, o bico fica pressurizado a 250 °C, escorrendo filamento que suja a escova e a peça.
   - *Solução:* Retração imediata de **`1.0 mm` a 1800 mm/min (`G1 E-1.0 F1800`)** e restauração instantânea da temperatura de impressão (`M104 S[target_temp]`).
2. **Risco de Colisão do Eixo Z:** Quando a peça está nas primeiras camadas (`Z < 30 mm`), a mesa levantada pode colidir com o mecanismo móvel da escova de silicone ao acionar a rampa mecânica.
   - *Solução:* **Salvaguarda inteligente de Z suave** — rebaixa a mesa para `Z=30 mm` a suaves `10 mm/s` (`F600`) em campo aberto (`Y=215`), realiza a limpeza e retorna com precisão para a camada exata.
3. **Looping de Corte do CFS (Extrusora Nova):** Em extrusoras novas com molas mais rígidas, o microswitch de corte `PA8` muitas vezes falha no acionamento, travando o firmware em repetições infinitas de corte.
   - *Solução:* Calibração do offset de corte para **`cut_pos_offset = 1.8`** (+0.5 mm).

---

## ⚙️ Parâmetros Calibrados

| Parâmetro | Valor Calibrado | Finalidade |
| :--- | :---: | :--- |
| **Zona Segura Frontal** | `Y = 215.0 mm` | Campo aberto para descida e subida do eixo Z |
| **Centro dos Pinos de Silicone** | `X = 168.0 mm` | Ponto de alinhamento central da escova |
| **Engate Traseiro da Rampa** | `Y = 226.3 mm` | Pressiona a rampa e ergue a escova contra o bico |
| **Amplitude de Esfregada** | `X = 165.0` a `171.0` | Curso lateral de 6 mm sobre as cerdas de silicone |
| **Ciclos de Limpeza** | `6 ciclos (12 passadas)` | Velocidade de 60 mm/s (`F3600`) |
| **Retração Anti-Gotejamento** | `1.0 mm` a 30 mm/s | Despressuriza o bico antes da escovação |
| **Velocidade de Salvaguarda Z** | `10 mm/s (F600)` | Movimento suave sem trancos ou perda de passos |
| **Offset da Alavanca de Corte** | `cut_pos_offset = 1.8` | Acionamento firme do sensor de corte na nova extrusora |

---

## 🚀 Como Instalar

### 1. Inserir a Macro no Klipper (`box.cfg`)

Abra o arquivo `box.cfg` pelo painel web do Fluidd / Mainsail / CrealityOS e adicione a macro `CFS_SAFE_WIPE` (disponível em [`macros/cfs_safe_wipe.cfg`](macros/cfs_safe_wipe.cfg)):

```gcode
[gcode_macro CFS_SAFE_WIPE]
description: Limpeza calibrada nos pinos de silicone do CFS com salvaguarda de Z
gcode:
  G90
  M400
  
  ; 1. Salva a altura Z real da peca
  {% set orig_z = printer.toolhead.position.z|float %}

  ; 2. Retracao rapida de 1.0mm anti-gotejamento
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G91
    G1 E-1.0 F1800
    G90
    M400
  {% endif %}

  ; 3. Restaura a temperatura de impressao (cancela os 250C da purga)
  {% set target_temp = params.TARGET_TEMP|default(232.0)|float %}
  {% if target_temp > 150.0 %}
    M104 S{target_temp}
  {% endif %}

  ; 4. Avanca para a zona segura
  G1 Y215 F6000
  M400

  ; 5. Salvaguarda de Z suave (se Z < 30mm)
  {% if orig_z < 30.0 %}
    G1 Z30 F600
    M400
  {% endif %}

  ; 6. Alinha com o centro da escova
  G1 X168 F6000
  M400

  ; 7. Engate traseiro erguendo a escova
  G1 Y226.3 F3000
  M400

  ; 8. 6 ciclos (12 passadas) de 6mm a 60 mm/s
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X165.0 F3600
  G1 X171.0 F3600
  G1 X168.0 F3600
  M400

  ; 9. Desengata para a frente
  G1 Y215 F6000
  M400

  ; 10. Restaura Z para a camada exata a 10 mm/s
  {% if orig_z < 30.0 %}
    G1 Z{orig_z} F600
    M400
  {% endif %}
```

### 2. Chamada Automática nas Trocas do CFS

Ainda em `box.cfg`, certifique-se de chamar a macro `CFS_SAFE_WIPE` logo após as purgas do CFS:

```gcode
[gcode_macro BOX_LOAD_MATERIAL_WITH_MATERIAL]
gcode:
  # ... rotina padrao de purga ...
  BOX_EXTRUDE_MATERIAL
  BOX_EXTRUDER_EXTRUDE
  ; --- CHAMA LIMPEZA CALIBRADA ---
  CFS_SAFE_WIPE
  RESTORE_POSITION
```

### 3. Configuração no OrcaSlicer / Creality Print

No seu fatiador, acesse as configurações de cada filamento (**Filament Settings -> Filament Start G-code**) e insira:

```gcode
CFS_SAFE_WIPE TARGET_TEMP=[nozzle_temperature]
```

Isso garante que ao final da purga nativa (que roda a 250 °C), o bico retorne imediatamente para a temperatura correta de impressão (ex: 220 °C para PLA, 230 °C para ABS, 245 °C para PETG).

---

## 🧪 Teste a Seco

Para testar a cinemática e o alinhamento da escova sem precisar imprimir, execute no console do Klipper:

```gcode
TESTE_LIMPEZA_BICO
```

A cabeça irá simular o fim da purga, descer a mesa para Z=60mm com segurança, realizar as 12 passadas na escova e retornar ao centro da mesa.

---

## 🤝 Contribuições

Contribuições são muito bem-vindas! Se você testou na **K1 Max** ou **K1C**, ou calibrou parâmetros para bicos de alto fluxo / materiais especiais como TPU, abra uma [Issue](https://github.com/Demolidor31br/creality-k1-cfs-nozzle-wiper/issues) ou envie um Pull Request.

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja [`LICENSE`](LICENSE) para mais detalhes.

Desenvolvido por **Ivan Marcos ([@Demolidor31br](https://github.com/Demolidor31br))**.
