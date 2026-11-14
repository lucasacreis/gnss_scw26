# GNSS SCW26 — Satellite Orbit Explorer

O projeto é desenvolvido no notebook [Satellite_Orbit_Explorer.ipynb](./Satellite_Orbit_Explorer.ipynb). Ele carrega dados de órbita no formato GP/OMM, calcula trajetórias aproximadas para vários satélites e exporta gráficos, arquivos KML e um relatório.

## Requisitos

- Python no ambiente virtual `.venv`
- `matplotlib` e `requests` (a primeira célula do notebook instala essas dependências)
- O arquivo de dados `orbit_workshop_inputs.zip`

## Preparar e executar

1. Abra `Satellite_Orbit_Explorer.ipynb` no VS Code e selecione o kernel `.venv`.
2. Coloque `orbit_workshop_inputs.zip` na pasta do notebook.
3. Use **Restart and Run All** para executar as células na ordem.
4. Ao final, encontre os resultados em `satellite_orbits/output/`.

A etapa de entrada procura estes três arquivos dentro de `satellite_orbits/input/`: `orbital_elements.json`, `data_manifest.json` e `world_map.txt`. Se estiverem faltando, o notebook procura um ZIP `orbit_workshop_inputs*.zip` na pasta do projeto, extrai os arquivos e para com erro claro se o arquivo estiver incompleto ou não for encontrado no tempo limite.

## Etapas do notebook

1. **Configuração:** importa as bibliotecas, cria as pastas de entrada e saída e define as constantes físicas e o intervalo padrão de 24 horas, com passo de 5 segundos.
2. **Carregamento dos dados:** lê os registros GP/OMM, valida a presença dos campos orbitais necessários e carrega o manifesto da fonte. O nome, o identificador NORAD e o epoch de cada registro são exibidos.
   - **Extensão opcional — CelesTrak ao vivo:** `LIVE_GROUP` vem como `None`; nesse modo, o notebook mantém o snapshot do workshop e não faz uma requisição. Se um grupo for definido, os dados são obtidos do CelesTrak, validados e guardados em cache por grupo por até duas horas. A extensão seleciona até quatro registros em ordem crescente de NORAD e atualiza `source_note` com o horário UTC da modificação do cache; `data_manifest` não é alterado.
3. **Conversão dos elementos:** transforma os campos OMM em elementos orbitais usados nos cálculos, incluindo conversões de graus para radianos, movimento médio para rad/s e cálculo do semieixo maior.
4. **Janela temporal:** cria uma grade UTC inclusiva de 24 horas, começando à meia-noite da data do epoch mais recente. Também mostra a diferença entre o início da janela e o epoch de cada registro e sinaliza registros com mais de sete dias.
5. **Propagação:** resolve a equação de Kepler, calcula posição e velocidade e converte as coordenadas orbitais para latitude, longitude e altitude. A célula inclui verificações numéricas para GMST, resíduos do solver e, quando presente, a longitude do SES-5.
6. **Resumo orbital:** calcula período, perigeu, apogeu e velocidades mínima e máxima, além de verificar o período do Galileo 29 quando esse registro está nos dados.
7. **Gráficos:** lê as linhas costeiras de `world_map.txt`, desenha os ground tracks e salva gráficos de trajetória e velocidade.
8. **KML e relatório:** exporta trajetórias e uma animação KML para Google Earth, grava um relatório com a fonte, janela, resumo e elementos orbitais, valida os KMLs gerados e reúne os arquivos em um ZIP.

## Arquivos gerados

O notebook cria `satellite_orbits/output/` e gera:

- `ground_tracks.png` — trajetórias sobre o mapa.
- `velocity_timeseries.png` — velocidade ao longo do tempo.
- `orbit_tracks_ecef.kml` — trajetórias com altitude.
- `orbit_animation.kml` — trajetórias animadas com controle de tempo.
- `orbit_report.txt` — fonte, janela, resumo orbital e elementos utilizados.
- `satellite_orbit_outputs.zip` — pacote dos cinco arquivos acima.

Abra qualquer KML no Google Earth Pro; para reproduzir a animação, use o controle de tempo.

## Limitações do modelo

Os registros GP contêm elementos médios SGP4, mas o notebook os usa em um modelo educacional de dois corpos com Terra esférica. As posições não são previsões orbitais de precisão. A latitude é geocêntrica e o Google Earth usa WGS-84, portanto as posições exibidas podem diferir; a idade do epoch também pode aumentar o erro de propagação.
