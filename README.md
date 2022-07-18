# Análise de Posições na NBA

Projeto Final realizado na disciplina de Álgebra Linear no Bacharelado em Ciência da Computação na UFRJ.


## Autor

- Gustavo Vilares Mariz de Oliveira
- DRE: 121073784



## Objetivo

![alt text](https://www.gannett-cdn.com/-mm-/dfff082d1e4931b30569ae37195b6862a6a8ef8a/c=0-361-2915-2008/local/-/media/2018/05/22/USATODAY/USATODAY/636625868623447717-AP-APTOPIX-Heat-Bucks-Basketball-39255807.JPG?width=660&height=373&fit=crop&format=pjpg&auto=webp)

Esse projeto tem como objetivo inicial analisar as estatísticas de jogadores da NBA, 
presentes na base de dados do Kaggle "1991-2021 NBA Stats", selecionando a temporada de 2019, e, 
por meio das técnicas de Álgebra Linear aprendidas em sala de aula, 
desenvolver um modelo que receba estatísticas de jogadores e indique a posição
que mais se encaixa com as estatísticas do jogador. 



## Um Pouco sobre Basquete!

![alt text](https://i0.wp.com/www.fiveminutesspare.com/nba/wp-content/uploads/sites/21/2019/04/Basketball-positions-2-1024x712.jpg?resize=650,400)

No basquete, temos 5 posições clássicas:
- Armador(1) : Responsável por organizar o time, levando a bola pro campo adversário, realizando os passes necessários e pontuando se necessário.
- Ala-Armador(2) : Responsável pela pontuação e defesa no perímetro.
- Ala(3) : Responsável pela defesa do perímetro e infiltração na área do adversário, pontuando bastante e também abrindo espaço para o ala-armador pontuar. Também tem papel em rebotes e assistências.
- Ala de Força(4) : Papel semelhante ao do pivô, atuando fortemente na defesa e no ataque do garrafão.
- Pivô(5) : É a figura principal no garrafão, sendo responsável por rebotes e "post-ups". Também tenta trazer defensores para si para abrir espaço para outros jogadores ofensivos.

Considerando esses diferentes papéis, espera-se que por meio da análise das estatísticas, 
seja possível identificar padrões e reconhecer a posição ideal de determinado jogador.

## De onde parte a análise

De início, a análise das estatísticas vai partir do conceito da distância de um vetor para um plano.
Buscando trabalhar em 3 dimensões, escolhi 3 estatísticas que melhor poderiam distinguir um jogador para
definir sua posição. Assim, por meio de uma análise de correlação feita no Python e do meu conhecimento prévio
sobre basquete, selecionei o total de cestas de 3 pontos realizadas (3P), o total de assistências (AST) e o total de bloqueios feitos (BLK).
Já com os dados separados, apliquei um método de padronização dos dados já que esses se apresentam em escalas diferentes.
Dessa forma, é possível formar vetores em R3 para cada jogador e colocá-los em um gráfico, já com os dados padronizados.

![image](https://user-images.githubusercontent.com/82617621/179436831-a614720c-0e7e-45e9-a726-a713889518a8.png)

Com os jogadores e suas posições identificados, 
formei uma planilha separada para cada posição com as respectivas estatísticas selecionadas:
    
    #Todos os jogadores
    est = ['3P', 'AST', 'BLK']
    todos = dados_pad[['Pos','3P', 'AST', 'BLK']]

    #Planilha dos armadores
    a  rmadores = dados_pad[dados_pad['Pos'] == 'PG']
    armadores = armadores[est].transpose()

    #Planilha dos ala-armadores
    ala_armadores = dados_pad[dados_pad['Pos'] == 'SG']
    ala_armadores = ala_armadores[est].transpose()

    #Planilha dos alas
    alas = dados_pad[dados_pad['Pos'] == 'SF']
    alas = alas[est].transpose()

    #Planilha dos alas de força
    ala_forca = dados_pad[dados_pad['Pos'] == 'PF']
    ala_forca = ala_forca[est].transpose()

    #Planilha dos pivôs
    pivos = dados_pad[dados_pad['Pos'] == 'C']
    pivos = pivos[est].transpose()

Ao formar uma matriz com os jogadores para cada posição, utilizei a técnica SVD para formar um plano para cada posição.



## Uso do SVD e a distância para o Plano

O SVD, ou Singular Value Decomposition, em suma, permite que eu fatore a matriz desejada em 3,
sendo uma ferramenta capaz de realizar uma aproximação de menor posto.

![image](https://user-images.githubusercontent.com/82617621/179448265-ff77d963-e35d-4048-b19f-cd73210ff9ec.png)

Em (b) é ilustrado como concretizar tal aproximação.

No exemplo usado dos dados da NBA em 2019, U é uma matriz ExE (sendo E o número de estatísticas usadas),
V é uma matriz JxJ (sendo J os jogadores) e temos no meio uma matriz diagonal.
Para realizar uma aproximação de posto k, só é preciso pegar k linhas de U, k colunas de V e k itens da diagonal da matriz central.

Desse modo, realizando uma aproximação para posto 2, é possível compreender que U possui os dois vetores que formam o plano de uma posição
e V seleciona o quanto de cada vetor de U é preciso para se aproximar de determinado jogador
(realiza uma combinação linear dos vetores de U).

Considerando um exemplo para os Armadores:

Realizando essa combinação linear, tenho a projeção no plano dos armadores para cada armador.
Logo, para comparar alguém de fora dos armadores, separei os dados do jogador a ser comparado 
e o adicionei no fim da lista de armadores antes de aplicar o SVD.
Desse modo, ao aplicar o SVD, gero a projeção desse jogador no plano dos armadores.
Assim, só é preciso calcular a distância do vetor do jogador original para sua projeção,
calculando o quão diferente esse jogador é de um armador clássico.


## Testes Iniciais
Para testar o quão eficiente é o modelo criei duas funções para executá-lo e criar as comparações.
A função compara_posicao() recebe como argumentos o nome do jogador a ser comparado e a posição que se deseja comparar.
Ela retorna a distância do vetor desse jogador para o plano formado pelo SVD para a posição escolhida.

Utilizando a compara_posicao(), também desenvolvi a descobre_posicao(). Essa função recebe como argumento o nome de um jogador e
retorna um vetor com uma comparação entre ele e todas as posições, sendo a menor distância a indicada pelo modelo para o jogador.

    descobre_posicao('Chris Paul') = 
    [0.1761565533050851, 
    1.7247781358191405, 
    1.837916547887539, 
    1.2968786399652512, 
    1.6873632320759946]

Sendo Chris Paul um armador clássico, que cria jogadas pelo passe e aproveita os bons arremessos,
temos o item 1 como a menor distância e, portanto, o modelo reconheceu que ele é um armador.

    descobre_posicao('LeBron James') =
    [0.5731366446225711,
    1.2314090006199674,
    1.84502031000131,
    1.5502159342930657,
    1.7865681874210981]

Temos alguns casos que o modelo erra claramente. Um importante é o Lebron James. Sendo ele
um dos melhores jogadores da história do basquete, ele atua de diversas maneiras na quadra. Sua posição original
é a de ala ou ala de força, porém, o modelo o indica como armador. Isso acontece provavelmente porque nos times em que ele joga
ele tem a maior posse de bola e é o jogador mais envolvido em jogadas. Assim, quanto mais versátil
é um jogador, mais difícil é de determinar com exatidão sua posição.


## Acertos Exatos

Com as funções de teste desenvolvidas, preparei um código para definir o quão preciso foi o modelo.
Nesse caso, não considerei nenhuma margem de erro, então só foram considerados acertos se o modelo recomendou exatamente a posição
que o jogador ocupou na temporada de 2019.

![image](https://user-images.githubusercontent.com/82617621/179442123-ce08319c-a4b0-4191-b25c-30adc7bca9e6.png)

Com a execução desse teste, cheguei em 162 acertos exatos e 368 erros, numa amostragem de 530 jogadores.

## Acertos Aproximados

Ao notar a falta de precisão exata do modelo, resolvi testá-lo de forma menos exata.
Para isso, passei a considerar um acerto toda vez que o modelo selecionava uma posição
que pode atuar de forma semelhante à original do jogador.

![image](https://user-images.githubusercontent.com/82617621/179442326-a4e2c5d1-fa21-486b-8a13-7e8cc4aaeb48.png)

Com a execução desse teste, cheguei em 345 acertos e 185 erros em uma amostragem de 530 jogadores.

A precisão ainda não é ideal, mas já se torna uma evolução e traz a tona o que foi observado anteriormente:
jogadores versáteis ou muito diferenciados vão fugir do modelo. Analisando a lista dos 185 jogadores errados temos que,
em grande parte, são jogadores que atuam de forma diferente da tradicional exercida pela sua posição.
Entre eles temos pivôs e alas de força com alta carga de cestas de 3 pontos, algo típico de armadores e ala-armadores,
estrelas do jogo como o LeBron e o Jayson Tatum, que fazem um pouco de tudo durante o jogo, armadores
que focam na infiltração e não no arremesso de 3, entre outros.

## Um modelo melhorado
Considerando a constante evolução do basquete, uma nova classificação de posições poderia melhorar a precisão do modelo.

Logo, por meio da biblioteca SKLearn, reavaliei os dados dos jogadores e os dividi
em 4 clusters por meio das estatísticas definidas:

    dados_cluster = dados_pad.drop(columns=['Pos'])
    dados_cluster = dados_cluster[['3P', 'AST', 'BLK']]
    kmeans_model = KMeans(n_clusters = 4, random_state = 1)
    kmeans_model.fit(dados_cluster)
    labels = kmeans_model.labels_

Assim, atingi uma divisão bem mais definida dos jogadores no gráfico de dispersão do que a
vista anteriormente no início dessa análise.

![image](https://user-images.githubusercontent.com/82617621/179444682-fa3cd8bf-357c-4164-a7c9-1dbad9095fef.png)

Utilizando a mesma lógica do SVD utilizada anteriormente, defini uma função para encontrar os
planos de cada cluster e ver o quanto um jogador está próximo desses. Assim desenvolvi a função
descobre_posicao_cluster(), que, utilizando a função anterior compara_posicao(), recebe o nome
de um jogador como argumento e retorna um vetor com as distâncias desse jogador para cada plano de cluster.

Desse modo, fiz a busca por acertos exatos, onde o modelo identifica o exato cluster original do jogador.

![image](https://user-images.githubusercontent.com/82617621/179445249-43d9d93c-648d-44e8-8acd-0aba6dec4183.png)

Dessa vez, tive 264 acertos e 266 erros em uma amostragem de 530 jogadores.

Os resultados já foram mais satisfatórios do que com as posições originais, porém,
ainda não foi possível adivinhar com exatidão os jogadores e o cluster/posição ao qual pertencem.

A clusterização também não permite um modelo de acertos aproximados, já que não se sabe a proximidade da atuação no jogo de cada cluster.
## Conclusões

Portanto, por meio das observações estatísticas e visuais, percebe-se que é muito difícil 
definir com exatidão a posição de um jogador. Por que isso acontece?

- Jogadores Estrelas:
    
    Como dito anteriormente, aquele jogador que brilha mais que os outros na quadra, participando de todas as jogadas
    defensivas e ofensivas, acaba não tendo uma posição definida, já que suas estatísticas podem apontar para muitas posições diferentes.

- Positionless Basketball

    Atualmente, existe um movimento de descaracterização das posições do basquete. Cada vez mais os termos aqui abordados
    têm sido menos usados e substituídos por Front-Court e Back-Court, justamente pelo entendimento de que
    cada vez mais os jogadores exercem diversos papéis durante a rotação do jogo e, portanto, não se prendem a um papel epecífico.

- Aumento dos arremessos de 3 pontos

    Parte da minha análise envolve a consideração dos arremessos de 3 pontos. Anteriormente, esse tipo
    de jogada ofensiva era característica de armadores e ala-armadores. Hoje em dia, cada vez mais todas as
    posições têm realizado o arremesso de três, em que temos hoje pivôs especializados nisso, como Karl-Anthony Towns,
    que foi campeâo do campeonato de arremessos de 3 pontos. O que torna o basquete ainda mais "positionless".

Tendo em vista esses obstáculos, acredito que o modelo por meio do acerto aproximado traz um resultado interessante,
sendo definido por essa ideia de Front-Court e Back-Court, trazendo uma taxa de acerto bem mais alta.
Assim, fica claro que cada vez mais as posições definidas antigamente se afastam da realidade do jogo, o qual se torna cada vez mais sem posição.



