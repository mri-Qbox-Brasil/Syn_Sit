# Syn_Sit — Manual

Sistema de sentar, deitar e encostar: cadeiras e camas do mundo viram assentos com target, mais "sentar em qualquer lugar" e "encostar na parede".

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Comandos e teclas](#comandos-e-teclas)
5. [Assentos por objeto (`config.lua`)](#assentos-por-objeto-configlua)
6. [Assentos customizados (`customize_seats.lua`)](#assentos-customizados-customize_seatslua)
7. [Assentos por zona (`polyseats.lua`)](#assentos-por-zona-polyseatslua)
8. [Sentar em qualquer lugar](#sentar-em-qualquer-lugar)
9. [Encostar na parede](#encostar-na-parede)
10. [Modo debug — criando assentos](#modo-debug--criando-assentos)
11. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | Declarado como `dependency` no manifest. Usa `lib.registerContext`, `lib.showContext`, `lib.showTextUI` |
| `ox_target` | Sim (se `Config.TargetSytem = 'OX'`) | `addModel`, `addPolyZone` |
| `qb-target` | Sim (se `Config.TargetSytem = 'QB'`) | `AddTargetModel`, `AddPolyZone` |

O recurso é client-only e não usa framework (QBCore/ESX).

---

## Instalação

1. Copie a pasta `Syn_Sit` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure Syn_Sit
   ```
3. Ajuste `Config.TargetSytem` em `config.lua` para o target que você usa (`'OX'` ou `'QB'`).
4. **Desligue o modo debug em produção** — `Config.Debug` vem como `true` no repositório. Veja [Modo debug](#modo-debug--criando-assentos).

Não há SQL nem itens de inventário.

O `fxmanifest.lua` declara `escrow_ignore` para todos os arquivos Lua e da NUI — ou seja, eles ficam abertos mesmo se o recurso for empacotado.

---

## Configuração

Opções gerais no topo de `config.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Config.TargetSytem` | string | Sim | `'OX'` (ox_target) ou `'QB'` (qb-target) |
| `Config.SmoothCancelSit` | number | Sim | ID de controle para levantar com transição suave. Padrão `32` (W) |
| `Config.QuickCancelSit` | number | Sim | ID de controle para levantar imediatamente. Padrão `58` (G) |
| `Config.OpenmenuWhileSitting` | number | Sim | ID de controle que reabre o menu de animações enquanto sentado. Padrão `203` (Espaço) |
| `Config.StartChairCamera` | number | Sim | ID de controle que ativa a câmera de cadeira. Padrão `23` (F) |
| `Config.AllowChairCamera` | bool | Sim | Habilita a câmera livre enquanto sentado (útil quando o ângulo padrão fica ruim) |
| `Config.AddHelpText` | bool | Sim | Exibe o texto de ajuda ao sentar |
| `Config.WhileSittingHelpText` | string | Sim | Texto de ajuda exibido enquanto sentado |
| `Config.WhileSittinginfoFunction` | function | Sim | Função que mostra o texto de ajuda. Por padrão usa `lib.showTextUI` e some após 5 s. Troque aqui para usar sua própria notificação |
| `Config.WhileSittinginfoFunctionCancel` | function | Sim | Função que esconde o texto de ajuda (`lib.hideTextUI`) |
| `Config.WhileSittinginfoFunction_IsOpenCheck` | function | Sim | Função que informa se o texto de ajuda está aberto (`lib.isTextUIOpen`) |
| `Config.OxContextMenuTitle` | string | Sim | Título do menu de contexto de animações (concatenado com o nome do assento — deixe um espaço no fim) |
| `Config.OxContextName` | string | Sim | Prefixo do nome das opções do menu |
| `Config.OxContextLabel` | string | Sim | Label da entrada que abre o menu de opções |
| `Config.OxTargetLabel` | string | Sim | Label da opção de target no objeto (ex.: `'Sentar '`) |
| `Config.AllowHeadingChange` | bool | Sim | Permite girar o personagem sentado com `A` e `D` |
| `Config.Debug` | bool | Sim | Habilita os comandos de criação de assento e os markers de debug. **Deixe `false` em produção** |
| `Config.DebugMarkerColor` | `vector4` | Sim | Cor RGBA dos markers do modo debug |
| `Config.AnimDescriptions` | table | Sim | Título e descrição de cada scenario/animação exibidos no menu. **É aqui que se faz a tradução** |
| `Config.SitAnyWhereAnims` | array | Sim | Animações e scenarios disponíveis no comando `/sit` |
| `Config.objects` | array | Sim | Assentos ligados a modelos de objeto do mundo |

`Config.AnimDescriptions` aceita duas formas:

```lua
-- Scenario: chave = nome do scenario
["PROP_HUMAN_SEAT_CHAIR"] = { title = "Assento de Cadeira", description = "Animação de assento para cadeiras." },

-- Animação: chave = dicionário, sub-chave = nome da animação
['anim@gangops@morgue@table@'] = {
    ['ko_front'] = { Title = 'Lie Still', description = 'play dead.' }
},
```

---

## Comandos e teclas

| Comando / Tecla | Permissão | Descrição |
|---|---|---|
| `/sit` | Nenhuma | Entra no modo "sentar em qualquer lugar": posiciona um clone translúcido do personagem e confirma o assento |
| `J` | Nenhuma | Encostar na parede. Keybinding `_SynSit_Lean` (remapeável em Configurações > Atalhos do FiveM) |
| `W` | Nenhuma | Levantar com transição suave (`Config.SmoothCancelSit`) |
| `G` | Nenhuma | Levantar imediatamente (`Config.QuickCancelSit`) |
| `F` | Nenhuma | Alternar a câmera de cadeira (`Config.StartChairCamera`, exige `Config.AllowChairCamera`) |
| `Espaço` | Nenhuma | Reabrir o menu de animações enquanto sentado (`Config.OpenmenuWhileSitting`) |
| `A` / `D` | Nenhuma | Girar o personagem sentado (exige `Config.AllowHeadingChange`) |

Comandos adicionais existem **apenas com `Config.Debug = true`** — veja [Modo debug](#modo-debug--criando-assentos).

---

## Assentos por objeto (`config.lua`)

`Config.objects` liga um modelo de objeto do mundo a uma lista de animações de assento. O target é adicionado ao modelo (`addModel` / `AddTargetModel`).

```lua
{ objName = "v_med_bed2", Animations = {
    { right_left_X = 0.0, forward_backwards_Y = 0.0, up_down_z = 1.2, Heading = 290.0, anim = 'WORLD_HUMAN_PICNIC' },
    { right_left_X = 0.0, forward_backwards_Y = 0.0, up_down_z = -0.6, Heading = 0.0, dict = 'anim@gangops@morgue@table@', anim = 'ko_front' },
    { anim = 'PROP_HUMAN_SEAT_CHAIR', right_left_X = 0.2, forward_backwards_Y = 0.0, up_down_z = 0.4, Heading = 90.0, IsSittingAnim = true, skipExitScene = false },
} },
```

| Campo | Tipo | Descrição |
|---|---|---|
| `objName` | string ou hash | Nome do modelo ou hash numérico do objeto |
| `Animations` | array | Lista de animações oferecidas no menu para esse objeto |
| `multiSeat` | array de string | Nomes dos lugares quando o objeto tem mais de um assento (ver abaixo) |
| `points` | array de `vector3` | Alternativa a `objName`: cria uma polyzone em vez de target por modelo |

Campos de cada animação:

| Campo | Tipo | Descrição |
|---|---|---|
| `anim` | string | Nome do scenario (`PROP_HUMAN_SEAT_*`) ou da animação dentro de `dict` |
| `dict` | string | Dicionário de animação. Se ausente, `anim` é tratado como scenario |
| `right_left_X` | number | Offset lateral em relação ao objeto |
| `forward_backwards_Y` | number | Offset para frente/trás |
| `up_down_z` | number | Offset de altura |
| `Heading` | number | Heading do personagem sentado, relativo ao objeto |
| `IsSittingAnim` | bool | Marca a animação como de sentar (afeta a saída) |
| `skipExitScene` | bool | Pula a cena de saída ao levantar |

### Objetos com múltiplos assentos

Quando `Animations` é uma tabela indexada (`[1] = {...}, [2] = {...}`) e existe `multiSeat`, cada índice vira um lugar independente e o nome vem de `multiSeat[i]`:

```lua
multiSeat = {
    "lado Direito",
    "Centro",
    "lado Esquerdo",
}
```

---

## Assentos customizados (`customize_seats.lua`)

`Config.custom_seats` tem exatamente o mesmo formato de `Config.objects`. É o lugar para registrar props de MLOs e mobília que não é nativa do GTA V, mantendo `config.lua` limpo. O arquivo já traz exemplos (cadeira de taverna, banquinho do Vanilla Unicorn, bancos em L e em U do Burger Shot).

---

## Assentos por zona (`polyseats.lua`)

`Config.PolySeats` cria assentos em coordenadas fixas do mundo, sem depender de um objeto — útil para muretas, degraus, o letreiro de Vinewood etc.

| Campo | Tipo | Descrição |
|---|---|---|
| `Seats` | array de `vector3` | Coordenada de cada lugar da zona |
| `points` | array de `vector3` | Polígono da zona de target |
| `thickness` | number | Espessura da polyzone |
| `distance` | number | Distância máxima de interação |
| `multiSeat` | array de string | Nome de cada lugar (mesma ordem de `Seats`) |
| `Animations` | tabela indexada | Animações por lugar, no mesmo formato de `Config.objects` |
| `debug` | bool | Desenha a zona |
| `objName` | `false` | Sempre `false` nas polyseats |
| `ZoneID` | `nil` | Preenchido em runtime pelo recurso |
| `options` | table | Opções extras de target |

---

## Sentar em qualquer lugar

O comando `/sit` clona o personagem com transparência, tocando a primeira animação de `Config.SitAnyWhereAnims`. O clone acompanha a mira da câmera; o jogador escolhe onde e como sentar antes de confirmar.

Controles do modo de posicionamento (exibidos na tela):

| Tecla | Ação |
|---|---|
| Clique esquerdo (`INPUT_ATTACK`) | Confirma e senta |
| Clique direito (`INPUT_AIM`) | Reseta a posição |
| `E` (`INPUT_TALK`) | Próxima animação da lista |

Cada entrada de `Config.SitAnyWhereAnims` tem:

| Campo | Tipo | Descrição |
|---|---|---|
| `dict` | string | Dicionário de animação, ou o nome do scenario quando `anim = false` |
| `anim` | string ou `false` | Nome da animação. `false` significa que `dict` é um scenario |
| `zOff` | number | Ajuste de altura da animação |

---

## Encostar na parede

A tecla `J` (comando `_SynSit_Lean`, remapeável) faz o personagem encostar na parede mais próxima. O recurso:

1. Detecta a parede e a normal dela.
2. Sorteia um conjunto de animações compatível com o gênero do ped (`mp_m_freemode_01` / `mp_f_freemode_01`).
3. Escolhe a animação de entrada correta conforme o ângulo da parede.

Só funciona com o jogador a pé, vivo, parado (não correndo, nadando ou em combate corpo a corpo). Os conjuntos de animação ficam no topo de `client/lean.lua`.

---

## Modo debug — criando assentos

Com `Config.Debug = true`, ficam disponíveis:

| Comando | Descrição |
|---|---|
| `/CreateObjectSeat` | Mira em um objeto e cria interativamente um assento para o modelo dele. Gera o bloco de config pronto para colar |
| `/Createpolygonseat` | Cria uma polyzone de assento no mundo, ponto a ponto. Gera o bloco de config pronto para colar |
| `/SaveMyHeading` | Exibe o heading atual do personagem, já arredondado, para colar na config |
| `/fixchair` | Reposiciona o personagem no assento atual usando os offsets da config — para ajuste fino sem restart |

A saída dos comandos é exibida na NUI (`html/index.html`). O arquivo `client/seat_maker.txt` traz notas do autor sobre o processo de criação.

**Deixe `Config.Debug = false` em produção**: os comandos ficam abertos a qualquer jogador e os markers de debug são desenhados na tela.

---

## Estrutura de arquivos

```
Syn_Sit/
├── client/
│   ├── client.lua           — registro do target (ox/qb), menus de contexto, entrada/saída do assento, câmera de cadeira
│   ├── utils.lua            — funções auxiliares (raycast de câmera, arredondamento)
│   ├── lean.lua             — encostar na parede: conjuntos de animação, detecção de parede, keybinding J
│   ├── sitanywhere.lua      — comando /sit: clone translúcido, seleção de animação e posicionamento
│   ├── debug.lua            — comandos de criação de assento (só com Config.Debug)
│   └── seat_maker.txt       — notas do autor sobre como criar assentos
├── config.lua               — opções gerais, descrições de animação, animações do /sit e Config.objects
├── customize_seats.lua      — Config.custom_seats: assentos de props de MLO / não nativos
├── polyseats.lua            — Config.PolySeats: assentos por coordenada, sem objeto
├── html/
│   ├── index.html           — painel usado pelo modo debug para exibir coordenadas e blocos de config
│   ├── init.js              — recebe as SendNUIMessage do debug
│   └── jquery.js
└── fxmanifest.lua
```
