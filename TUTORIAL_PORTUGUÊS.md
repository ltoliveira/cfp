Este é um guia para os usuários do código que desenvolvemos. Entendendo as funções, o usuário poderá testar em outras regiões metropolitanas do Brasil.
Dividmos o tutorial de acordo com os scripts na [plataforma do GEE](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project).
Importante ressaltar que as variáveis/atributos usados variam contextualmente. O conjunto de dados que usamos foi derivado de uma longa pesquisa com especialistas locais.
Alguns atributos provavelmente serão usados em outros contextos como o Modelo de elevação digital ou distância de áreas naturais, mas a interpretação dos valores possivelmente mudará visto que as características da precariedade local também mudarão.

**Obs.: As palavras 'variável' e 'atributos' são usadas de forma intercambiável nesse tutorial.**

## 1a_prep_AGSNbuffer
Este código apresenta duas funções. Primeiramente, ele carrega as duas camadas principais do modelo: a camada com a extensão da área de estudo e a camada dos assentamentos informais (AGSN - aqui usados como indicador de precariedade espacial local).
Além disso, o código exclui possíveis pixels de áreas não precárias normalmente presentes nas bordas da amostra. Esse pixels podem gerar ruído e por isso são excluídos com uma função 'buffer'.
Antes de determinar o tamanho dessa zona de exclusão ('buffer'), o usuário precisa definir a escala de análise, isto é, a resolução final do modelo - aqui usamos o formato raster, em grade retícula.
O tamanho da buffer deverá ser metade do valor da resolução do pixel, pois assim a função excluirá os pixels que estão com menos da metada da sua áreas dentro da amostragem do AGSN.
No exemplo, usamos uma grade de 20x20m, portanto, a zona de exclusão é 10m. 
O código exporta o resultado como um [GEE asset](https://developers.google.com/earth-engine/guides/asset_manager), que o usuário pode acessar pela aba 'asset' no Code Editor.

**Favor checar todas as vezes que 'scale' ou 'SCALE' for mencionado nos scripts abaixo, dentro de alguma função ou para exportar como asset. O valor de 20 referente a resolução final deve ser ajustado de acordo com as necessidades do usuário.**

## 1b_prep_OSMmerge 
Este código processa e combina os dados vetoriais OpenStreetMap, os quais precisam ser baixados na plataforma [HOTOSM](https://www.hotosm.org/) fora do catálogo GEE e importados como assets.
Como a região metropolitana de São Paulo é bem extensa, a plataforma exige o download em quatro lotes para abranger toda a área de estudo.
O código combina os lotes por tipo de atributo (ponto, linha ou polígono) e exporta como asset como mostrado abaixo.
Se a região de interesse (ROI) for maior e precisar de mais lotes que quatro, outras variáveis precisarão ser adicionadas representando o(s) lote(s) extra(s). 
Se a ROI requerer soment um lote, esse script pode ser ignorado. 
![image](https://user-images.githubusercontent.com/101252763/198277399-e6dc5bc3-825b-4c42-b65e-7e3e75e59acd.png)

## 1c_prep_RuasSemSaida
Este código foi criado para identificar e extrair ruas sem saída dos dados vetoriais de linha do OSM, especificamente buscando encontrar os pontos finais da ruas. 
Este processo não é automatizado em nenhum software SIG, o que faz a nossa abordagem inédita. A única mudança que o usuário precisa fazer é, por testes, substituir a variável 'osmSimplified' por 'osmRandomColumn'. 
Isto vai depender da complexidade e da completude da malha viária OSM da área de estudo. 
![image](https://user-images.githubusercontent.com/101252763/198278537-5ec1e604-cf82-4b80-addc-f16323095432.png)
O script exporta o resultado da camada vetorial (pontos) que vai ser usada no script '2c_input_KDE' para a Estimativa de Densidade Kernel (KDE) dos pontos sem saída.

## 2a_input_RSdados
Aqui o usuário pode extrair atributos de dados de sensoriamento remote, respectivamente Índice de Vegetação por Diferença Normalizada (NDVI), Modelo de Elevação Digital (DEM), Declive, Contagem populacional, Luzes noturnas (VIIRS-DNB).
O usuário possivelmente precisará ajustar a data de aquisição das imagens de satélite (*FilterDate'*) de acordo com sua necessidade. Como a camada AGSN foi produzida em 2019, mantivemos a consistência temporal.
![image](https://user-images.githubusercontent.com/101252763/198280244-d5d1dff6-6a6f-4cba-97d9-191ea8742b1a.png)

A partir deste código, todos os resultados são exportados e usados como dados de entrada do modelo. 

## 2b_input_GLCM
Este código extrai cinco métricas de textura derivadas da Matriz de co-ocorrência de níveis de cinza (GLCM, em inglês). O GEE já possui uma função que calcula essas métricas, por isso o código só faz o processament da banda pancromática do Landsat e roda a função. O usuário só precisa, se necessário, ajustar o raio do kernel.
Aqui, usamos 5 (em pixels, portanto 100m), considerando a estrutura espacial da região metropolitana de São Paulo. Dependendo da área de estudo, um valor menor de raio seja mais eficiente para mostrar o número de quadras urbanas, ou um número maior para um contexto de crescimento espraiado. O usuário precisa tomar essa decisão.
![image](https://user-images.githubusercontent.com/101252763/198281734-59ebb130-fc1a-411c-9f99-0a65bd4a899d.png)

## 2c_input_KDE
Este script calcula a densidade dos pontos finais das ruas sem saída extraídos previamente no script *'1c_prep_deadends'*. O usuário só precisa ajustar o raio do kernel e para isso, pode-se utilizar dos resultados visualizos no mapa.

## 2d_input_MSI
Este código traz uma abordagem inovadora para calcular o índice de Forma (MSI, em inglês), uma conhecida métrica da paisagem. Da camada de mancha urbana global (WSF, em inglês, World Settlement Footprint) de 2019, calculamos o MSI para cada pixel e combinamos o valor por unidade setorial do AGSN. 
O dado do WSF também precisou ser adquirido em lotes, dois no caso. O usuário pode precisar de mais. Se sim, após o download e importação como asset na conta GEE, é preciso carregar como uma nova variável no código e incluí-la(s) na função do 'mosaic'.
![image](https://user-images.githubusercontent.com/101252763/198286258-a4b69ba2-6602-4630-8af5-11ed6e6b4a63.png)

Além disso, o usuário precisa ajustar a escala e o raio dos 'reducers' de cada filtro. Usamos os filtros para reduzir possíveis ruídos, mas o usuário precisa checar o impacto dos filtros em assentamentos pequenos ou muito grandes. Use a visualização no mapa para identificar o raio mais adqueado.
![image](https://user-images.githubusercontent.com/101252763/198286797-d77121bb-9c9d-41ec-aff6-b9c2ae87c1e5.png)

## 2e_input_GISdados
Este código calcula a distância euclideana entre atributos vetoriais de dados OSM como rede viária, corpos hídricos, áreas vegetadas, unidades de saúde, etc. Após a importação dos dados OSM, nada precisa ser ajustado além da escala de análise (no código 20m). O código pode ser adaptado para incluir outros atributos, como distânia a áreas industriais ou redes de esgoto por exemplo. Isso dependerá da disponibilidade (e completude) do dado e do contexto estudado. 

## 3_input_Inspeção
Este código visualize todos os dados de entrada (variáveis/atributos) preparados para o modelo. Aqui, o usuário deve inspecionar possíveis fontes de inconsistências ou padrões estranhos. Se algum dado for adicionado, o usuário precisa incluir o parâmetro de visualização necessário.
![image](https://user-images.githubusercontent.com/101252763/198287743-7997dbe7-1a99-47d6-a14f-4df54b882baa.png)

## 3_input_Zscore 
Este script calcula o z-score, medindo a distância entre o valor do pixel e a média de todos os pixels usando desvio padrão. Isto ajuda a encontrar e remover valores atípicos que podem atrapalhar a performance do modelo. Esta é a versão menos automatizada onde o usuário precisa escolher a variável (linha 45 do código) e  limite a regra do z-score. O padrão usado é -3 < z < 3, aqui usamos -3 < z < 4. O usuário pode definir isso testando diferentes regras e visualizando no histograma (aba console). 
![image](https://user-images.githubusercontent.com/101252763/198289015-be2596e5-76b0-4bf1-9c84-6d7ed9fbedba.png)

## 4_model_Filtro
This code implements the k-means algorithm, visualize the results and applies a post classification filter to deal with the noise derived from the own classification process and to deal with the voids derived from the z-score calculation.
The first thing the user can "play with" is to removing and including new variables. By adding // to the line code the algorithm do not read that variable. The user also need to change (if there are new variables or variables removed) from the image collection. 
![image](https://user-images.githubusercontent.com/101252763/194076455-19454e7c-bd12-4dcc-a20d-87b452756dfb.png)

Under the training part, the user can change the number of clusters. This is a very important step that has no proper decision methods besides trial and error. Other programming languages offer tools for this e.g.,Python: elbow method, but we could not apply into this javascript yet. 
![image](https://user-images.githubusercontent.com/101252763/194077592-5941794a-c5e4-4b0d-b78d-999e83e151be.png)

To visualize the graphs, the user will need to remove or include the variables name and rename the bands accordingly. 
![image](https://user-images.githubusercontent.com/101252763/194078171-dda79b2f-6553-4305-ad95-83f87baa95f7.png)

And remove or include the number of graphs according to the number of clusters. Besides changing on line 172-175 of the code, the user should remove all the lines regarding the creation of the histograms that are not selected. For instance, we used 4 four clusters, but if there are only 2 clusters for a certain context, the histograms 3 and 4 must be erased. The user can also select the variable they would like to see on the graph, even only one at a time.
![image](https://user-images.githubusercontent.com/101252763/194078332-657da266-5ee5-4820-95af-34281e41a05a.png)

Finally, we used spatial filters for a better visualization of the output map. The user can change the kernel radius (line 265) and check the influence on the final result by visualizing it on the map.
![image](https://user-images.githubusercontent.com/101252763/194079649-8bb5df27-6e03-4372-aaa6-215f7206e9ac.png)

The screenshot below shows the final map result and the first cluster map.
![image](https://user-images.githubusercontent.com/101252763/194081061-8f86305a-7e4c-40b4-9a21-3e4070865316.png)
Scrooling down the console tab, the other histograms can be seen and by clicking on the gray arrow on the right corner of the graph, it opens in another tab with download options in different formats.
![image](https://user-images.githubusercontent.com/101252763/194081642-1c409717-2e03-4ce8-b2b3-3970e6f94b2d.png)
