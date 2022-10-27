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
Este código implementa o algoritmo k-médias, visualize os resultados (com mapa e histogramas) e aplica filtros pós classificatórios a fim de lidar com possíveis ruídos derivados do próprios processo de clusterização e do cálculo do z-score. 
O usuário pode testar incluir e remover variáveis e observar os resultados. Se adicionar // a linha do código, o algoritmo não lerá a variável. Caso adicione ou retire alguma variável, o usuário precisa modificar na coleção de imagens que é o dado principal de entrada do model. 
![image](https://user-images.githubusercontent.com/101252763/198291576-396edcfd-6cb1-46fa-bc2a-1b2dd3dc4420.png)

Na seção de treinamento do modelo, o usuário pode modificar o número de clusters e também testar o efeito no resultado com a visualização no mapa e histogramas. Por ser modelo não supervisionado, além do conhecimento local, é preciso usar método qualitativo para tomada de decisão. Outras linguagens de programação oferecem ferramentas para cálculo quantitativo, por exemplo em Python pode-se aplicar o elbow method, mas em Java não é possível. Estamos trabalhando nisso. 
![image](https://user-images.githubusercontent.com/101252763/198292229-c2772081-55b2-4a32-9631-6626bfb857e0.png)

Para visualizar os gráficos o usuário precisa remover ou incluir o nome das variáveis usadas e renomear as bandas de acordo.
![image](https://user-images.githubusercontent.com/101252763/198292492-df56a914-6f07-4903-913a-2b5e707b0e23.png)

E remover ou incluir o número de gráficos de acordo com o número de clusters definido (linhas 170-173 do código). Por exemplo, se só três clusters forem selecionados para certo contextos, a linha do histograma 4 deve ser apagado. O usuário também pode selecionar a(s) variável(is) que se deseja ver no gráfico, mesmo que seja uma de cada vez. 
![image](https://user-images.githubusercontent.com/101252763/198293268-2deb202e-5194-4c83-bcf7-c616d7db7a8f.png)

Por fim, usamos filtros espaciais para melhorar a visualização do resultado final. O usuário pode mudar o raio do kernel (linha 264) e checar a influência no resultado final visualizando no mapa.
![image](https://user-images.githubusercontent.com/101252763/198294180-dffef6e6-9558-40b4-b228-712a43334107.png)

Abaixo, pode-se ver uma captura de tela que mostra um zoom do resultado final e o histograma do cluster número 1 na aba console. 
![image](https://user-images.githubusercontent.com/101252763/194081061-8f86305a-7e4c-40b4-9a21-3e4070865316.png)

Os demais histogramas estão logo abaixo e eles podem ser vistos amplamente em uma outra janela se clicar na pequena seta cinza no canto direito do gráfico. O GEE oferece vários formatos de download dos gráficos o que é bastante conveniente também. 
![image](https://user-images.githubusercontent.com/101252763/194081642-1c409717-2e03-4ce8-b2b3-3970e6f94b2d.png)
