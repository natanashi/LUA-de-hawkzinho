# LUA de hawkzinho

Um pequeno sistema solar em WebGL: uma lua, um planeta azul de continentes
vermelhos e um sol de verdade, num céu escuro cheio de estrelas. Arquivo único,
sem dependências e sem servidor — é só abrir o `index.html` no navegador.

![captura](docs/preview.png)

## Como usar

| Ação | O que faz |
| --- | --- |
| Roda do mouse | Zoom (0.6× a 14×), ancorado no ponteiro |
| Clicar e arrastar | Gira o espaço e a lua junto; ao soltar, desliza e desacelera |
| Pinça (toque) | Zoom |

Não há botões nem texto na tela, por design.

O sol começa fora da vista — é assim no céu de verdade: com a lua gibosa, o sol
está a ~130° de distância; os dois nunca caberiam juntos num campo de visão
estreito. Arraste o céu para encontrá-lo e veja a fase da lua mudar ao vivo
conforme ele se move. Alinhe o sol atrás da lua e você ganha um eclipse total,
com o anel da coroa ao redor do disco escuro.

## Como funciona

Tudo é desenhado por um único fragment shader, sem texturas nem modelos.

- **A lua** é uma esfera com relevo gerado proceduralmente: crateras de um campo
  celular 3D, em oitavas que vão sendo ligadas conforme o zoom, de acordo com o
  tamanho que um pixel ocupa na superfície. O detalhe acompanha a ampliação sem
  custar caro de longe. As normais saem do gradiente da altura (bump mapping por
  gradiente de superfície), com uma amostra de altura por pixel.
- **Os mares** são planícies de lava jovens: engolem as crateras grandes e ficam
  mais escuros e lisos. É esse contraste que impede a superfície de virar um
  campo uniforme de bolhas.
- **A iluminação** mistura Lambert com Lommel-Seeliger, que é o que dá à Lua real
  aquele aspecto de disco chapado em vez de bola sombreada nas bordas.
- **As estrelas** vivem numa grade em faces de cubo, em camadas: as mais fracas
  aparecem conforme você aproxima, mantendo a densidade na tela e o custo limitado.
  O raio delas é fixo em pixels, então continuam pontos em qualquer zoom.
- **As estrelas cadentes** são desenhadas em espaço de tela, com cabeça compacta
  e rastro que afina e some. Duas trilhas independentes dão cerca de uma a cada
  7 segundos.
- **O sol** é um disco com escurecimento de limbo real (mais brilhante no centro,
  como o Sol de verdade) e uma coroa aditiva. Toda a luz da cena sai dele: as
  fases da lua e do planeta são pura geometria, não pintura.
- **O planeta** tem oceanos azuis, continentes vermelhos com litoral definido,
  calotas polares, nuvens que deslizam sobre o chão, reflexo do sol no mar
  aberto e borda azul de atmosfera (Rayleigh).

## A física (dados reais, tempo comprimido)

- Raio do planeta = 3.667 raios lunares — a razão Terra/Lua da NASA (6371/1737 km).
- Eixo do planeta inclinado 23.4°, como o da Terra.
- A lua é travada por maré: gira sobre si na mesma taxa em que o céu roda,
  então o planeta nunca sai da face que a encara — como a Lua real, que mostra
  sempre o mesmo lado para a Terra.
- Dia do planeta : mês lunar = 1 : 27.32, a razão real (24 h : 27.32 dias).
- Tempo comprimido ~300× (1 s ≈ 5 min): o dia do planeta leva ~4.8 min na tela
  e o mês lunar ~2.2 h. Nada gira rápido.
- Distâncias comprimidas para a composição, como em qualquer planetário; o sol
  aparece maior que os 0.53° reais para não virar um ponto ofuscante.

O giro do céu e da lua é guardado em quatérnions, para acumular arrasto em
qualquer direção sem deriva. A lua recebe um ângulo maior que o céu para o mesmo
arrasto: ela é uma esfera de raio 1 a 7 unidades de distância, então o mesmo
ângulo varre bem menos pixels na tela. O fator `1/MOON_TAN` faria a superfície
acompanhar o cursor exatamente, como girar um globo com a mão — mas o arrasto
costuma acontecer longe do disco, no céu aberto, e ali essa taxa fica nervosa.
`MOON_DRAG` segura a rédea: com 0.33, as estrelas seguem o cursor 1:1 e a
superfície anda um terço disso.

Todo movimento tem a velocidade dividida pelo zoom, para o cenário andar igual
perto ou longe.

## Requisitos

WebGL 1 com a extensão `OES_standard_derivatives` (ou seja, qualquer navegador
dos últimos ~10 anos). A resolução de render cai sozinha se a placa não der
conta, e nunca sobe de volta — uma escada só de descida não oscila.
