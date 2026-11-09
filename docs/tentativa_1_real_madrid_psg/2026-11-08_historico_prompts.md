# Histórico de prompts e relatório do desenvolvimento

Data: 2026-11-08. Projeto: Prática 3 de Visão Computacional (CESAR School), análise de futebol com YOLO.
Repositório: https://github.com/LucasRafaelG/An-lise-de-Futebol-com-YOLO.

Este arquivo tem duas partes: (1) os prompts que mudaram o projeto, em ordem; (2) um relatório do que foi feito. Ficaram de fora as dúvidas (perguntas de "como funciona", "o que é", "qual o próximo passo") e as mensagens que não alteraram nada no projeto. Os textos dos prompts são os originais; o que está entre colchetes é contexto acrescentado.

## Parte 1: prompts que geraram trabalho

1. [Depois de perguntar como baixar o vídeo e se o ffmpeg precisava ser instalado]
   > instale o ffmpeg e me instrua a como rodar o programa e colocar o link do video para baixar

2. [Comando usado para baixar o trecho do jogo, conferido antes de rodar]
   > esta certo assim?
   > `.\venv\Scripts\python.exe vc-20262-p3\baixar_video.py "https://www.youtube.com/watch?v=TRYsA-ON9-A" --inicio 40 --fim 44 --saida vc-20262-p3\videos`

3. > então me diga quais os proximos passos, APENAS MONTE O PASSO A PASSO NO CHAT NÃO ALTERE NADA

4. > adiccione nesse github todos os arquivos,
   > [instruções do GitHub para criar o repositório: `git init`, `git remote add origin https://github.com/LucasRafaelG/An-lise-de-Futebol-com-YOLO.git`, `git push -u origin main`]

5. > voce pode enviar as pastas saidas e videos com os arquivos delas dentro?

6. > os trechos de replay são:
   > 0:40 a 1:00, 1:30 a 1:48, 2:05 a 2:19, 3:30 a 3:43
   > você mesmo pode fazer essa remoção?

7. > pode commitar e dar push

8. [Depois da explicação do bola_kalman.py]
   > voce pode fazer?

9. [Resposta "sim" à pergunta: montar o comando de calibração com os pontos clicáveis do frame0. Isso levou à criação do jogo2, começando em 0:05]
   > sim

10. [Depois de clicar os 5 pontos na janela de calibração]
    > rodei e marquei

11. > comite e de push

12. > monte o proximo comando mas não execute, meu amigo vai rodar porque o pc dele é melhor

13. > monte um prompt para o claude dele entender todo o contexto e ja rodar esse comando

14. [Colando a saída da etapa da câmera rodada pelo Helder: 7050 quadros, 3720 quadros-chave, 1448 falhas, 15 cortes]
    > isso aqui?

15. [Para aplicar a correção da etapa da câmera]
    > roda

16. > pare de rodar

17. > reverta

18. > faça o prompt para helder rodar

19. > crie um arquivo com esse scrip

20. [Colando a resposta do Claude do Helder: 98,6% dos quadros inválidos; diagnóstico de limite de inliers e da área espelhada]
    > saida do claude de helder

21. [Colando a segunda resposta do Claude do Helder: 89,7% inválidos; a regra de razão de inliers cortava a cadeia]
    > saida do claude de helder

22. > me de um prompt com todo o passo a passo para fazer esse projeto, do inicio ao fim, o claude deve fazer todo o projeto e para quando tiver algo para o ser humano fazer ou se der algum erro

23. > faça as correções necessárias para essa m*** parar de dar erro

24. > commite e de push

25. > faça um prompt para helder rodar no claude dele que de seguimento ao projeto

26. > crie o arquivo para eu alterar

27. > coloquei os que eu consegui anotar, fiz essas anotações com base no tempo da gravação do rastreamento em jogo1

28. > faça o que voce puder gerar

29. > quero que você faça um md com o historico de todos os prompts mas deixe de fora os que foram duvidas e mensagens que não serviram para o desenvolvimento do projeto, tambem relate tudo o que foi feito ate aqui em uma parte separada

## Parte 2: relatório do que foi feito

### Ambiente e vídeo

- Ambiente Python (`venv`) com `vc-20262-p3/requirements.txt`; ffmpeg instalado com `winget install Gyan.FFmpeg`.
- Trecho baixado: minuto 40 ao 44 do vídeo (Real Madrid 3×1 PSG, oitavas da Champions 2017/18), 1280×720 a 30 fps, 7200 quadros.

### Detecção e rastreamento

- `detectar_rastrear.py` com `yolov8m.pt`, `imgsz` 1280, só CPU: 0,68 quadro/s, cerca de 2 h 56 min para os 7200 quadros (pasta `saidas/jogo1`).
- O trecho tem replays, closes e ângulos atrás do gol. O ROTEIRO pede trecho sem cortes; o problema apareceu numa folha de quadros a cada 20 s.

### Remoção de replays e pasta jogo2

- Novo script `remover_trechos.py`: remove os replays do `tracks.csv` sem refazer o YOLO, guardando o original em `tracks_completo.csv`. Replays removidos: 0:40–1:00, 1:30–1:48, 2:05–2:19, 3:30–3:43 (65 s).
- O primeiro quadro não tinha marcos do campo para calibrar. O script ganhou `--inicio` e `--destino`, e foi criada a pasta `saidas/jogo2`, começando em 0:05 (quadro com linha do meio e círculo central), reaproveitando o rastreamento.

### Bola

- `bola_kalman.py` no jogo2: YOLO com candidata de bola em 48,6% dos quadros; cobertura final de 60,2% após o filtro de Kalman.

### Calibração e câmera

1. Primeira calibração com 5 cliques no frame0 (linha do meio e círculo). Foi corrigido um bug: com 5 pontos o código usava RANSAC, que descartava um clique bom (erro de 2,11 m); passou a usar mínimos quadrados com menos de 8 pontos (erros de 0,07 a 0,56 m). Foram criados os marcos `circulo_esq` e `circulo_dir` em `campo.py`.
2. A etapa `homografia.py camera` original produziu matrizes degeneradas: depois de ~30 s, todos os quadros caíam no mesmo ponto do campo.
3. Duas correções foram enviadas ao Helder sem teste e falharam (98,6% e 89,7% de quadros inválidos), por limites de inliers mal calibrados e por uma regra de razão de inliers. O Claude do Helder diagnosticou o problema e corrigiu um bug de área espelhada.
4. Correção final, testada em trechos curtos antes da rodada completa:
   - CLAHE e FAST 5 aumentaram os pontos ORB do gramado de ~90–600 para ~2000–2700 por quadro;
   - toda matriz passa a ser validada (`_plausivel`);
   - o vídeo é dividido em segmentos (planos contínuos de câmera), já que os pontos da grama não se repetem depois de um corte;
   - cada quadro clicado ganha a própria calibração (`calibrar --quadro N`), e cada quadro usa a calibração mais próxima do mesmo segmento, porque a cadeia deriva ~2–4 m num pan de 20 s;
   - novo subcomando `conferir`, que desenha as linhas do campo sobre o vídeo.
5. Calibração dos segmentos: 18 quadros calibrados (5 por mim e 13 pelo Helder, que também corrigiu o desenho das linhas para homografias de escala negativa). Resultado: 48,0% dos quadros com câmera calibrada, 83,3% das detecções de jogadores e 49,6% das posições da bola com posição em metros; 14 de 15 painéis do `conferir` alinhados.

### Métricas, validação e eXP

- `metricas.py`: posse 35,1% (time 0) contra 64,9% (time 1); 30 passes detectados.
- 24 passes anotados à mão (`passes_anotados.csv`, convertidos para o relógio do jogo2). Validação: precisão 0,37, revocação 0,46, F1 0,41 (tolerância de 1 s). O erro dominante são passes falsos, concentrados no time 0.
- Novo script `experimentos.py`:
  - Q2.1: `yolov8n` contra `yolov8m`;
  - Q2.3: `imgsz` 640 contra 1280 e limiares de `conf-bola`;
  - Q3.3: `k=3`. Com `k=2`, as execuções que começam em 0:05 misturaram os dois times, porque um centróide ficou amarelo (o árbitro); com `k=3` o árbitro vai para "outros";
  - Q5.2: sensibilidade da posse ao raio de 1, 2 e 3 m.
- `xpass.py --what-if`: AUC do logístico 0,732, do heurístico 0,786 e do baseline 0,362. Ressalva: 19 dos 30 passes do modelo não casam com a anotação.
- Todos os números, por questão e com o arquivo de origem: `docs/2026-11-08_numeros_por_questao.md`.

### Repositório e colaboração

- Repositório criado e enviado ao GitHub. Os vídeos foram enviados por Git LFS e depois removidos do repositório, já que o ROTEIRO tira 10% por vídeo versionado; continuam no histórico.
- Prompts para o Claude do Helder (rodar a câmera, aplicar a correção, continuar o projeto) e um prompt do projeto completo em `docs/2026-11-08_prompt_projeto_completo.md`.
- Commits (mais recente primeiro): f1706df, 6976145, bea2574 e e8e9581 (Helder), c1dee07, 104bd96, 37fc55f (Helder), a3dc353, 51cafc5, b20245f, fbd1ee7, 9b06250.

### Pendências (humanas)

- README autoral com URL e intervalo do vídeo, comandos, respostas às questões e declaração de uso de IA.
- Q1.1, exemplos da Q2.3, Q3.1 (trocas de ID), exemplos da Q4.1, Q4.4 e a confirmação da direção de ataque (Q5.4).
- Push dos commits f1706df e 6976145, avisando o Helder antes: o `pull` apaga os `.mp4` da pasta dele.
