Eu sou professor de Godot, me ajude a planejar uma série de aulas com foco em Resources e Servers

Aula 1: Yggdrasil, não possui resources nem servers, na verdade ele é a Main Scene, porém extremamente importante para o resto do jogo, nessa aula ainda não é pra desenvolver a yggdrasil completa com o controle de save e etc, apenas a parte mais simples dela com máquina de estados, instanciamento de cenas e etc

Aula 2: SaveMemory, criando o Server e seus Resouces, aplicando isso ao jogo por meio da Yggdrasil

Aula 3: Inventory, server, resouce e Hud com drag in drop, exibição de cooldown, durabilidade e quantidade

Aula 4: TransitionCam, server, resouce, custom node, usando cameras virtuais que não extende diretamente de câmera, mas sim são pontos fixos ou móveis, que serve como âncora para a câmera do jogo, servindo justamente para poder transicionar tudo, cada câmera virtual contém suas regras e quando está ativado, a câmera principal transita para ela e assume os comportamentos definidos pela câmera virtual (resouce define comportamento)

Aula 5: Sistema de Habilidades (AbilitySystem), não possui server, possui custom node e resouces, ele é StateMachine + Effects + Skills
