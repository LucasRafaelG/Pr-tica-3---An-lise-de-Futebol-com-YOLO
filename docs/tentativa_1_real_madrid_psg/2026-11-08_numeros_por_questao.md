# Números por questão do ROTEIRO (jogo2)

Data: 2026-11-08. Material de apoio para escrever o README de entrega. **Não são as respostas**: o ROTEIRO exige análise e README autorais, com declaração de uso de IA. Aqui estão só os números gerados pelos scripts, o arquivo de onde cada um sai e observações para conferir. Todo número citado no README tem de bater com esses arquivos (penalidade de −20%).

Itens marcados com 👤 dependem de alguém assistir ao vídeo.

## Vídeo e execução

- URL: https://www.youtube.com/watch?v=TRYsA-ON9-A (Real Madrid 3×1 PSG, oitavas da Champions 2017/18).
- Trecho baixado: minuto 40 ao 44 do vídeo (`baixar_video.py --inicio 40 --fim 44`). Relógio do jogo no placar: 36:14 a 40:14 do 1º tempo.
- Execução analisada: `saidas/jogo2` = clipe a partir de 0:05 (vídeo 40:05–44:00). Começa 5 s depois do clipe porque o primeiro quadro não tinha marcos para calibrar (`remover_trechos.py --inicio 0:05`).
- Replays removidos (tempo do clipe): 0:40–1:00, 1:30–1:48, 2:05–2:19, 3:30–3:43 = 65 s (1950 quadros). Fonte: `saidas/jogo2/meta.json → trechos_excluidos_s`.

## Parte 1

- **Q1.2**: 1280×720, 30 fps (`saidas/jogo1/meta.json`). Clipe: 7200 quadros (240 s); jogo2: 7050 quadros, 5100 fora dos replays. Custo medido: `yolov8m`, `imgsz` 1280, só CPU: 0,68 quadro/s → 7200 quadros em ~2 h 56 min (`fps_processamento` em `saidas/jogo1/meta.json`).
- **Q1.1** 👤: justificativa da escolha. Fatos úteis: transmissão com cortes e replays (o ROTEIRO pede sem); uniformes com bom contraste (Real branco, PSG escuro); árbitro de amarelo.

## Parte 2 (subtrecho de 30 s: clipe 0:05–0:35, 901 quadros)

Fonte: `saidas/experimentos/experimento_modelos.csv` e `experimento_conf_bola.csv` (`experimentos.py modelos`).

| execução | modelo | imgsz | quadros/s | jogadores/quadro | % quadros com bola | candidatas de bola/quadro |
|---|---|---|---|---|---|---|
| m1280 | yolov8m | 1280 | 0,68 | 13,00 | 93,5 | 1,66 |
| n1280 | yolov8n | 1280 | 2,89 | 12,59 | 95,3 | 1,75 |
| m640 | yolov8m | 640 | 2,13 | 12,94 | 71,6 | 0,98 |

Observações:
- m1280 é o recorte (quadros 150–1050) da execução completa do jogo1, que também gravava o vídeo anotado; as outras rodaram com `--sem-video`. Os quadros/s variam com a carga da máquina: o mesmo `yolov8n` 1280 deu 2,89 numa execução e 3,67 em outra (`q3_yolov8n_k3`). Comparar quadros/s só em ordem de grandeza.
- "% quadros com bola" conta qualquer candidata, inclusive falsos positivos (até 5 por quadro). Neste subtrecho de câmera aberta passa de 90%; no jogo2 inteiro o YOLO teve candidata de bola em 48,6% dos quadros (Q4.1).
- **Q2.2** (dados): melhor candidata de bola por quadro tem lado mediano de 12,5 px (p10 9,0; p90 19,0) e área mediana de 154 px²; jogador: altura mediana 77 px, área mediana 2610 px². Confiança mediana: bola 0,44, jogador 0,81 (`saidas/jogo2/tracks.csv`).
- **Q2.3** (`--conf-bola`), % de quadros com alguma candidata acima do limiar:

| conf mín. | m1280 | n1280 | m640 |
|---|---|---|---|
| 0,1 | 93,5 | 95,3 | 71,6 |
| 0,2 | 83,9 | 88,2 | 58,7 |
| 0,3 | 72,7 | 80,0 | 48,1 |
| 0,5 | 55,8 | 56,5 | 34,0 |

  👤 Falta: exemplos visuais de falsos positivos da bola.

## Parte 3

- **Q3.1** 👤: contagem manual de trocas de ID em 1 minuto. Dado automático (inclui cortes de câmera, então não substitui a contagem): 477 IDs de jogador projetados no jogo2, com tempo mediano em campo de 1,5 s (`saidas/jogo2/metricas_jogadores.csv`).
- **Q3.2**: centróides BGR (`meta.json → times_bgr`). Execução principal (começa em 0 s, k=2): time 0 = (124, 117, 91), escuro (PSG); time 1 = (186, 218, 200), claro (Real). Proporção das detecções no subtrecho de 30 s: 53,1% / 46,3% / 0,7% não classificado (`experimento_times.csv`).
  Nas execuções de 30 s que começam em 0:05, o k=2 falhou: um centróide ficou amarelo (86, 191, 181), a cor do árbitro, e os dois times caíram no mesmo grupo (n1280: 85,6% / 9,0% / 5,4%; m640: 83,1% / 11,3% / 5,6%). O K-Means depende dos quadros usados para calibrar (`--calib-frames`, padrão 60).
- **Q3.3** (`q3_yolov8n_k3`, k=3): centróides (139, 128, 107) escuro, (224, 223, 202) claro, (85, 191, 180) amarelo → "outros". Proporção: 45,5% / 41,1% / 8,1% outros / 5,4% não classificado. O árbitro foi para "outros" e os times se separaram de novo.

## Parte 4

- **Q4.1** (`bola_kalman.py` no jogo2): YOLO com candidata em 3429 de 7050 quadros (48,6%), 5881 candidatas; aceitas após o gating: 2251; preenchidas pelo Kalman: 1991; cobertura final: 60,2% (inclui os 1950 quadros de replay no denominador).
  👤 Exemplos: trechos de 15 quadros só de Kalman (o máximo, `--max-lacuna`) começam, no tempo do `rastreado.mp4`, em 5,4 / 13,5 / 14,2 / 16,0 / 25,5 / 29,0 s. Ver no vídeo onde ele acerta e onde inventa.
- **Q4.2**: `saidas/jogo2/calibracao.json` (erro por marco) e `calibracao_grade.jpg`. São 18 quadros calibrados. Com 5 marcos (quadros 0, 285, 515, 1880, 4800, 5200, 5350), o maior erro é 0,82 m (`circulo_dir`, quadro 515); os do círculo são os maiores (pontas e cruzamentos de uma elipse, clicados a olho). Quadros com exatamente 4 marcos dão erro 0,00 m por construção: o ajuste é exato e o número não mede nada.
- **Q4.3**: o código usa `H_q · G_q⁻¹ · G_t`, a generalização de `H0 · G_t` com vários quadros calibrados (cada quadro usa o calibrado mais próximo do mesmo segmento). Deriva medida: juntar no mesmo ajuste os cliques do quadro 0 e do quadro 598 (20 s de pan depois) deixou até 3,76 m de erro (`meio_sup`). A calibração só do quadro 0, projetada na grande área aos 18 s, ficava visivelmente deslocada. Re-ancoragem: calibrar vários quadros (feito). Cobertura: 48,0% dos quadros com câmera calibrada; segmentos ≥ 2 s sem calibração: 21,0–23,1 s, 103,3–111,2 s, 147,9–150,9 s, 202,1–204,4 s (saída de `homografia.py projetar`). `camera_conferencia.jpg`: 14 de 15 painéis alinhados; em 68,1 s o círculo sai ~2 m deslocado.
- **Q4.4** 👤: conceitual.

## Parte 5

- **Q5.1** (`saidas/jogo2/metricas_times.json`, `metricas_jogadores.csv`): posse time 0 = 35,1%, time 1 = 64,9%; distância total: time 0 = 1812,4 m, time 1 = 2158,2 m. Maior velocidade máxima entre todos os IDs: 36,7 km/h. Cinco maiores distâncias: 72,4 m em 19,0 s (228 m/min), 66,8 m em 18,8 s (213), 63,0 m em 20,2 s (187), 58,0 m em 20,0 s (174), 52,8 m em 15,9 s (199); velocidades máximas desses: 19,8–23,6 km/h. A referência de ~120 m/min é a média de 90 min; aqui são trechos curtos de ataque, e a fragmentação dos IDs (Q3.1) também pesa.
- **Q5.2** (`saidas/jogo2/experimento_raio_posse.csv`, `figuras/sensibilidade_raio_posse.png`):

| raio (m) | posse t0 % | posse t1 % | passes | casados c/ anotação | precisão | revocação |
|---|---|---|---|---|---|---|
| 1 | 34,5 | 65,5 | 16 | 4 | 0,25 | 0,17 |
| 2 | 35,1 | 64,9 | 30 | 11 | 0,37 | 0,46 |
| 3 | 36,5 | 63,5 | 35 | 12 | 0,34 | 0,50 |

- **Q5.3** (`validar_passes.py --tolerancia T`, 30 passes detectados; anotação em `saidas/jogo2/passes_anotados.csv`, já no relógio do jogo2; anotada com precisão de ~1–2 s):

| anotados | tolerância | casados | precisão | revocação | F1 | acordo certo/errado |
|---|---|---|---|---|---|---|
| todos (24) | 1 s | 11 | 0,37 | 0,46 | 0,41 | 0,73 |
| todos (24) | 2 s | 12 | 0,40 | 0,50 | 0,44 | 0,75 |
| todos (24) | 3 s | 13 | 0,43 | 0,54 | 0,48 | 0,77 |
| só nos trechos analisados (19) | 1 s | 11 | 0,37 | 0,58 | 0,45 | 0,73 |
| só nos trechos analisados (19) | 2 s | 11 | 0,37 | 0,58 | 0,45 | 0,73 |
| só nos trechos analisados (19) | 3 s | 12 | 0,40 | 0,63 | 0,49 | 0,75 |

  5 anotados caem fora dos trechos com câmera calibrada (24, 104, 107, 109, 148 s), onde o programa não detecta passes. As linhas "só nos trechos analisados" foram calculadas com um CSV filtrado (mesmo script), não estão no repositório.
  Erro dominante: falsos positivos (30 detectados contra 19 reais nos trechos analisados), concentrados no time 0 (14 detectados, 6 anotados). Comparando só pelo tempo, os casamentos continuam 11, então a separação dos times não é a causa principal. Hipóteses a discutir: trocas falsas de posse quando a bola some ou "pula", e chutes ou divididas contados como passes errados (chutes não foram anotados).
- **Q5.4**: direção estimada `{0: esquerda, 1: direita}` (`metricas_times.json`). Indício: o time 1 é o de camisa clara (Real); no começo do trecho a camisa branca ataca a área da direita, onde está o goleiro de laranja do PSG. 👤 Confirmar no vídeo.

## Parte 6 (`xpass.py --what-if`, 30 passes, 53,3% certos; validação cruzada estratificada de 5 partes)

Fonte: `saidas/jogo2/resultados_xp.csv`, `jogadores_xp.csv`, `figuras/calibracao_xp.png`, `figuras/superficie_xp_f86.png`.

| modelo | AUC | Brier | Log-Loss | Avg Precision | média prevista |
|---|---|---|---|---|---|
| heurístico | 0,786 | 0,263 | 0,754 | 0,847 | 0,772 |
| baseline (distância) | 0,362 | 0,281 | 0,761 | 0,461 | 0,556 |
| logístico | 0,732 | 0,207 | 0,791 | 0,712 | 0,523 |

Taxa observada: 0,533. Coeficientes do logístico (features padronizadas): distância −0,974; adversários na linha 0,148; pressão no passador −0,354; espaço no destino 1,955; progressão x −0,119; distância do destino ao gol 0,175. What-if: ganho médio de eXP de 0,514.
Ressalva central para Q6.2: 19 dos 30 passes usados não casaram com nenhum passe anotado (Q5.3), então o modelo é treinado e avaliado sobre uma maioria de passes falsos, e com 30 exemplos a variação entre partições é grande. Juntar passes de outros grupos com `--extra` está previsto no ROTEIRO.

## Entrega: estado

- Vídeos fora do repositório: feito (commit 6976145; `*.mp4` no `.gitignore`). Continuam no histórico do Git; apagar de lá exige reescrever o histórico.
- `passes_anotados.csv`: feito.
- Falta (👤): README autoral com URL e intervalo, comandos, respostas e declaração de uso de IA; Q1.1, Q2.3 (exemplos), Q3.1, Q4.1 (exemplos), Q4.4, Q5.4 (confirmação) e a interpretação das demais.
