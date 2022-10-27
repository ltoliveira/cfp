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
Este código traz uma abordagem inovadora para calcular o índice de Forma (MSI, em inglês), uma conhecida métrica da paisagem. 
From the building footprint layer (WSF 2019) we calculated the Mean Shape Index formula for each pixel and combined the resulting values per settlement. 
The WSF datasets are also acquired by tiles and you might need to download more than we needed for our case study. So another variable should be imported to the user's assets, loaded into the code as a new variable and included in the mosaic. 
![image](https://user-images.githubusercontent.com/101252763/194071687-626b5d5c-7616-4ae7-a2e0-076c266cfb7b.png)
Besides that, the user will need to adapt the scale and the radius of the reducers of the applied filters (we used them for noise removal). Here we used 2 fos max reducers and 4 for min reducers. Use the displayed map to identify the most suitable radius, making sure to check how the filter impacts areas with small and large settlements.
![image](https://user-images.githubusercontent.com/101252763/194072106-faf2ab15-0f48-4dd1-b5a3-957089ac0fbe.png)

## 2e_input_GISdados
This code calculates the Euclidean distance to features (street network, water bodies, natural areas and urban facilities). Nothing must be added besides the scale of analysis after importing the OSM merged layers (line, points and polygons) to the script. 
This code can be adapted to include other features such as distance to industrial facilities, sewage plants or water wells. This depends on the context and the availability (and completeness) of the OSM data.

## 3_input_Inspeção
This code is basically to visualize all input data, inspecting source of inconsistencies or any weird patterns. If other input features are added, the user will need to look into standard visualization parameters for such dataset. 
![image](https://user-images.githubusercontent.com/101252763/194074306-aa9ad661-9bbd-4597-bd46-705c69721247.png)

## 3_input_Zscore 
This script calculates the z-score, measuring the distance between a data point and the mean using standard deviations. It helps finding and removing atypical values that can disturb the model performance. 
We created a manual and an automated version. In the manual version, the user will need to choose the variable on the top of the code to select the input feature, and on the end of the code for proper visualizing and exporting. 
![image](https://user-images.githubusercontent.com/101252763/194075380-180bdb3e-151c-4a71-83bf-e12a3b18be4f.png)
The user also need to select the z-score threshold. The standard rule is -3 < z < 3. 
![image](https://user-images.githubusercontent.com/101252763/194075562-53d7d920-3007-4732-b1a5-e0340a0595a3.png)

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
