# Pipeline ETL para Análise de Radiação de Fundo da Internet (IBR)

Pipeline completo de análise de IBR do projeto Cloud Telescope, desenvolvido como parte do
Trabalho de Conclusão de Curso de Matheus Baron Lauritzen (Ciência da Computação, UNIVALI), sob
orientação do Prof. Fabricio Bortoluzzi.

As etapas 1 e 2 (captura via TShark e enriquecimento geográfico/AWS) são do *pipeline* ETL
original do grupo de pesquisa ([bortoluzzi/ibr-etl](https://github.com/bortoluzzi/ibr-etl)),
reproduzidas aqui para que o repositório seja autocontido. A etapa de sumarização
(`3_summarize_radiation.py`), também originada nesse *pipeline*, foi adaptada neste trabalho para
exportar os dados agregados também em formato JSON. As etapas 4 e 5 são inteiramente novas: a
etapa 4 converte esse JSON em arquivos `.dat` compatíveis com o Gnuplot, e a etapa 5 executa
automaticamente todos os *scripts* de plotagem `.plt`, produzindo o conjunto completo de 25
visualizações sem qualquer intervenção manual.

## Requisitos

- Python 3 com as dependências listadas em `requirements.txt` (`pip install -r requirements.txt`)
- [Gnuplot](http://www.gnuplot.info/) 6.x com o terminal `pngcairo`
- `tshark` (`sudo apt install tshark`), para a etapa 1
- Um banco de geolocalização por IP compatível com a biblioteca `geoip2` (ex.: GeoIP2-City.mmdb ou
  GeoLite2-City.mmdb, ver [MaxMind GeoLite2](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)),
  para a etapa 2

## Como executar

```bash
# 1. Captura: converte os arquivos .pcap.gz do diretório em um único output.csv.gz via TShark
./1_generate_tshark_csv.sh

# 2. Enriquecimento geográfico/AWS de cada IP de origem e destino
zcat output.csv.gz | python3 2_enrich_csv.py | gzip > output.csv.enriched.gz

# 3. Sumarização, gera summaries.json e tabelas em LaTeX
zcat output.csv.enriched.gz | python3 3_summarize_radiation.py > output.tex

# 4. Converte summaries.json nos arquivos .dat esperados pelos scripts Gnuplot
python3 4_generate_gnuplot_dat.py

# 5. Executa todos os scripts .plt e gera o conjunto completo de PNGs
python3 5_generate_graphics.py
```

As etapas 3 a 5 são independentes e desacopladas entre si: o conjunto de gráficos pode ser
regenerado a qualquer momento a partir de `summaries.json`, sem reprocessar o `output.csv`
original.

## Estrutura do repositório

- `1_generate_tshark_csv.sh` — captura via TShark (etapa 1, do *pipeline* original)
- `2_enrich_csv.py` — enriquecimento geográfico/AWS por IP (etapa 2, do *pipeline* original)
- `3_summarize_radiation.py`, `4_generate_gnuplot_dat.py`, `5_generate_graphics.py` — sumarização
  (adaptada) e as duas etapas novas de geração de visualizações
- `mask_geo_points.py` — filtra pontos geográficos sobre o oceano antes da renderização do mapa de
  calor (invocado automaticamente pelo Gnuplot via `system()`)
- `gerar_grafico_*.plt` — os 25 *scripts* Gnuplot, um por visualização
- `*.png` — exemplo do resultado de cada *script*, gerado sobre o Dataset~3 do Cloud Telescope
- `world_50m.dat`, `ne_50m_admin_0_countries.geojson` — dados geográficos de referência (contorno
  continental), usados pelo mapa de calor geográfico

## Contexto acadêmico

A descrição completa da metodologia, o critério de seleção dos 25 gráficos entre os estilos
oferecidos pelo Gnuplot, e a discussão de cada resultado estão documentados no TCC correspondente:

> LAURITZEN, Matheus Baron. **Desenvolvimento de um Pipeline Gnuplot para Análise de Eventos em
> Radiação de Fundo da Internet**. Trabalho de Conclusão de Curso (Graduação em Ciência da
> Computação) — Universidade do Vale do Itajaí, Itajaí, 2026. Disponível em:
> [`Matheus_Baron_LauritzenTCC3.pdf`](./Matheus_Baron_LauritzenTCC3.pdf).
