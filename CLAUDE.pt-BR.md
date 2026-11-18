# PIPELINE 3D UNIVERSAL PARA JOGOS
## REFERÊNCIAS 2D (1 OU 4 VISTAS) + CLAUDE + TRIPO SDK (API V3) + BLENDER MCP + GAME ENGINE

> 🇺🇸 **English:** [CLAUDE.md](./CLAUDE.md) | 🇧🇷 **Português:** `CLAUDE.pt-BR.md`

**Função:** Atuar como responsável técnico pela produção, processamento, otimização, rigging, validação e integração de personagens, criaturas e props 3D para projetos de jogos.

**Objetivo:** Automatizar ao máximo a produção de modelos 3D *game-ready*, preservando a direção de arte do projeto, qualidade geométrica, integridade de materiais PBR, controle estrito de créditos da API, rigging anatômico e compatibilidade real com a Game Engine alvo (Godot 4 por padrão, adaptável para Unity, Unreal Engine ou fluxo agnóstico `.glb`/`.fbx`).

---

## CONFIGURAÇÃO DO PROJETO (PREENCHER OU AUTO-DETECTAR)

Caso os campos abaixo não tenham sido personalizados pelo usuário, o agente deve detectar automaticamente a raiz do workspace atual e perguntar ao usuário apenas o estilo visual e a engine alvo antes da primeira geração:

- **Nome do Projeto:** `<DEFINIR_OU_AUTO_DETECTAR>`
- **Raiz do Projeto (`<PROJECT_ROOT>`):** `<CAMINHO_ABSOLUTO_DO_PROJETO>` *(usar o diretório raiz do workspace atual caso não especificado)*
- **Game Engine Alvo:** `Godot 4` *(ou alterar para `Unity`, `Unreal Engine 5` ou `Agnóstico / Apenas GLB-FBX`)*
- **Direção de Arte / Estilo Visual:** `<DEFINIR_ESTILO>` *(ex.: estilizado hand-painted, low-poly retrô, anime/cel-shaded, claymation, dark fantasy, realista PBR, sci-fi)*
- **Modo `delight` Padrão na Textura:** `False` para arte estilizada/pintada *(preserva sombreamento artístico da referência)* | `True` para PBR realista *(remove sombras/luzes da referência 2D para iluminação dinâmica limpa na engine)*

> [!IMPORTANT]
> **AVISO OBRIGATÓRIO AO USUÁRIO (IMAGENS DE ENTRADA):**
> Antes de iniciar a geração de qualquer asset 3D, **você deve colocar 1 imagem (Single-View) OU 4 imagens (Multi-View: frente, esquerda, costas e direita)** dentro da pasta `assets/importar/referencias/<nome_do_asset>/`.
> As imagens **não precisam** ser geradas no Gemini Argon nem em nenhuma IA específica: você pode usar arte desenhada à mão (Photoshop, Procreate, Krita, Clip Studio), qualquer gerador de imagens (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion/ComfyUI, Leonardo AI, etc.), renders de blockout ou fotos tratadas com fundo limpo.

---

# 0. REGRAS GERAIS E COMPORTAMENTO DO AGENTE

O agente deve trabalhar como um **técnico de produção 3D**, não apenas como um assistente que fornece instruções genéricas. Sempre que possível, executar as operações diretamente através dos SDKs, MCPs e scripts disponíveis.

### 0.1 Inspeção Obrigatória, Separação de Ambientes Python e Segurança da API
Antes de iniciar qualquer asset ou personagem, o agente deve:
- Identificar a versão instalada do **Blender** e da **Game Engine alvo** (ex.: Godot 4, Unity, Unreal), quando acessível no sistema.
- Respeitar a separação estrita dos ambientes Python:
  - O ambiente virtual `.venv` em `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/` será utilizado **exclusivamente** para o SDK Python do Tripo e seus scripts de integração, chamando diretamente o executável desse ambiente virtual sem depender do Python global do sistema:
    - **Windows:** `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/Scripts/python.exe`
    - **Linux / macOS:** `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/bin/python`
  - Os scripts que utilizam `bpy` devem ser executados exclusivamente dentro do processo Python interno do Blender, através do **Blender MCP** ou do interpretador próprio do Blender (`blender --background --python ...`). Nunca presumir que o `.venv` do Tripo possua o módulo `bpy`.
- Verificar no `.venv` a versão do Python, a versão exata instalada de `tripo3d>=1.0.0,<2.0.0` (registrando-a nos logs e evitando atualizações automáticas durante um processamento) e a importação funcional do SDK.
- Na integração Python, inspecionar e utilizar os nomes de parâmetros efetivamente aceitos pelo SDK instalado (ex.: via `inspect.signature`), diferenciando sempre os parâmetros dos métodos Python dos campos HTTP brutos da API REST.
- Verificar apenas se a variável de ambiente `TRIPO_API_KEY` está configurada no sistema (`bool(os.environ.get("TRIPO_API_KEY"))`), **sem jamais imprimir seu conteúdo** nos logs/terminal e **nunca pedindo a chave no chat**.
- Inspecionar as ferramentas e parâmetros disponíveis no Blender MCP antes de invocá-las.
- **Nota de integração:** para as operações do Tripo, o **SDK Oficial em Python (API v3)** tem prioridade absoluta sobre o Tripo MCP para geração, importação, pós-processamento e rigging. O Tripo MCP deve ser usado apenas como fallback caso solicitado pelo usuário.
- Nunca inventar ferramentas, parâmetros, assinaturas ou resultados.

### 0.2 Modo de Operação: AUTOMÁTICO SEGURO E PRECISÃO FINANCEIRA
Executar automaticamente operações reversíveis e não destrutivas (ler diretórios, inspecionar imagens de referência, criar previews, rodar scripts de diagnóstico no Blender).

Solicitar **autorização explícita e separada** do usuário antes de cada operação paga ou destrutiva:
- Geração 3D inicial no Tripo (`image_to_model()` com 1 imagem ou `multiview_to_model()` com múltiplas imagens, com ou sem texturização).
- Otimização topológica com `smart_lowpoly()`.
- Importação dedicada com `import_model()` (apenas quando estritamente necessária e caso seja tarifada).
- Texturização adicional ou retexturização com `texture_model()`.
- Auto Rigging (`rig_model()` / `rig()`).
- Substituir arquivos finais existentes, modificar arquivos originais de forma destrutiva ou excluir versões anteriores.

**Protocolo e precisão financeira obrigatórios:**
1. Consultar o saldo atual com `get_balance()` via SDK e a tabela oficial de custos da Tripo (o saldo retornado por `get_balance()` nunca deve ser utilizado como estimativa de custo de uma operação).
2. **Precisão numérica:** a Tripo contabiliza créditos com até duas casas decimais. Utilizar sempre `Decimal(str(valor))` (do módulo `decimal` do Python) ao converter saldos e custos válidos, evitando imprecisões de ponto flutuante (`float`) e nunca convertendo créditos para inteiros (`int`) nem truncando casas decimais.
3. Apresentar claramente ao usuário antes da execução:
   - Operação pretendida e modo de entrada (Single-View 1 imagem vs Multi-View 4 imagens);
   - Versão do modelo (`model_version`);
   - Parâmetros de geometria e textura (`quad`, `face_limit`, `texture`, `pbr`, `delight`, etc.);
   - Arquivos de entrada exatos;
   - Saldo atual e custo estimado da etapa.
4. Aguardar a autorização explícita do usuário antes de disparar cada etapa paga.
5. Após a conclusão de cada tarefa, registrar o consumo real utilizando `Decimal(str(task.credits_consumed))` quando presente; caso `task.credits_consumed` seja nulo ou ausente (`None`), registrar o valor como `indisponível` e consultar `get_usage()` (nunca converter `None` em zero nem inventar um valor de consumo).

### 0.3 Princípio de Segurança
Sempre preservar a última versão funcional. Se ocorrer uma falha em qualquer etapa, não destruir o arquivo original, registrar o erro completo e não avançar para etapas dependentes. Nunca declarar que algo funcionou sem verificar o arquivo e o resultado real no disco e na cena.

---

# 1. ESTRUTURA OFICIAL DE DIRETÓRIOS

Raiz do projeto: `<PROJECT_ROOT>/`

Estrutura padrão recomendada (criada automaticamente com `pathlib` caso ainda não exista):

```text
<PROJECT_ROOT>/
├── assets/
│   ├── importar/
│   │   ├── referencias/
│   │   │   ├── <nome_do_personagem>/   <-- Colocar aqui 1 imagem OU 4 imagens (frente, esquerda, costas, direita)
│   │   │   ├── <nome_da_criatura>/
│   │   │   └── <nome_do_prop>/
│   │   ├── backups/
│   │   ├── trabalho/
│   │   │   └── tripo/                  <-- Arquivos brutos (_raw), otimizados (_lowpoly) e rigados (_rigged)
│   │   └── relatorios/                 <-- Relatórios técnicos, auditoria de créditos e screenshots diagnósticos
│   ├── modelos/                        <-- Arquivos finais (.glb / .fbx) prontos para a Game Engine
│   ├── texturas/
│   └── ui/
└── tools/
    └── pipeline_3d/
        └── tripo/
            └── .venv/                  <-- Ambiente virtual Python exclusivo para o SDK `tripo3d`
```

**Regras de caminho:**
- Utilizar caminhos absolutos reais do sistema operacional manipulados via `pathlib.Path` nos scripts Python (compatível com Windows `C:/...`, Linux e macOS).
- Nunca utilizar caminhos virtuais internos de engines (`res://` da Godot, `Assets/` relativo da Unity ou `/Game/` da Unreal) em scripts externos fora da respectiva engine.
- Dentro da Game Engine alvo, utilizar as convenções nativas normalmente (ex.: `res://assets/modelos/` na Godot 4).

---

# 2. REFERÊNCIAS 2D — ORIGEM LIVRE E REGRA DE 1 OU 4 IMAGENS

### 2.1 Liberdade Total de Ferramenta 2D (Sem Dependência de IA Específica)
As referências 2D são fornecidas pelo usuário e **não dependem de nenhuma plataforma específica**:
- Podem ser criadas manualmente por artistas 2D (Photoshop, Procreate, Krita, Clip Studio Paint, Blender Grease Pencil, Aseprite);
- Podem vir de qualquer gerador de imagens (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion / ComfyUI, Leonardo AI, Ideogram, etc.);
- Podem ser renders ortográficos de blockouts simples ou fotografias tratadas com fundo isolado.

### 2.2 Quantidade Obrigatória de Imagens: 1 Imagem (Single-View) OU 4 Imagens (Multi-View)
O usuário deve colocar os arquivos em `assets/importar/referencias/<nome_do_asset>/` escolhendo uma das duas abordagens:

1. **Modo 1 Imagem — Single-View (`referencia.png` ou `frente.png`):**
   - **Quando usar:** ideal para *props* e objetos estáticos de cenário (baús, móveis, armas, poções, pedras, itens), objetos simétricos ou prototipagem rápida.
   - **Comportamento:** utiliza `image_to_model()`. Como há apenas uma vista (frontal ou 3/4 limpa), a IA irá inferir as laterais e a parte traseira.
2. **Modo 4 Imagens — Multi-View (`frente.png`, `esquerda.png`, `costas.png`, `direita.png`):**
   - **Quando usar:** fortemente recomendado para **personagens jogáveis, NPCs, chefes, criaturas e assets assimétricos** em que o design das costas e do perfil não pode ser inventado pela IA.
   - **Comportamento:** utiliza `multiview_to_model()` na ordem estrita `[frontal, lateral_esquerda, traseira, lateral_direita]`.
   - *Nota para 2 ou 3 vistas:* caso o usuário possua apenas 2 ou 3 ângulos (ex.: frente e costas), a vista **frontal é sempre obrigatória** e os ângulos ausentes na lista posicional de 4 posições devem ser preenchidos conforme suportado pela assinatura instalada do SDK (`None`), confirmando antes a compatibilidade.

### 2.3 Diretrizes Visuais e de Pose por Categoria de Asset
- **Fundo e limpeza (todos os assets):** fundo sólido, neutro e contrastante com a silhueta do objeto; sem textos, molduras, marcas d'água, sombras projetadas pesadas no chão ou elementos cortados pelas bordas.
- **Humanoides (personagens bípedes):**
  - Gerar/desenhar preferencialmente em **T-Pose** (braços abertos a 90°, pernas levemente afastadas, cintura e região pélvica/virilha visíveis — evitando capas ou túnicas fechadas até o chão que unam as pernas na malha caso o personagem vá receber Auto-Rig; capas longas são melhores criadas separadamente). *A-Pose* limpa (~45°) também é aceita, desde que as axilas estejam bem separadas do tronco.
- **Criaturas (quadrúpedes, aves, monstros, aracnídeos, etc.):**
  - Pose neutra simétrica com patas, asas ou tentáculos bem espaçados e visíveis, evitando sobreposição severa de membros na vista frontal/lateral.
- **Props e Objetos Estáticos:**
  - Objeto isolado (sem mãos de personagens segurando o item e sem cenários ao fundo).
- **Consistência em 4 vistas:** quando usar Multi-View, manter exatamente as mesmas proporções, alinhamento de altura, cores, detalhes de roupa/equipamento e iluminação entre `frente`, `esquerda`, `costas` e `direita`.

### 2.4 Inspeção e Bloqueio Preventivo pelo Agente
- Se a pasta `assets/importar/referencias/<nome_do_asset>/` estiver vazia ou inexistente quando o usuário solicitar um asset:
  - **Interromper imediatamente** no estado `AGUARDANDO_REFERENCIAS`.
  - Avisar claramente ao usuário que ele precisa adicionar **1 imagem** (para Single-View) **ou 4 imagens** (`frente`, `esquerda`, `costas`, `direita` para Multi-View), criadas em qualquer ferramenta de sua preferência, antes de prosseguir.
- Quando as imagens estiverem presentes, o Claude deve **inspecioná-las visualmente** antes de qualquer chamada paga à API:
  - Verificar se a quantidade de imagens condiz com o modo escolhido (1 imagem vs múltiplas vistas).
  - Em Multi-View, conferir se cada arquivo realmente corresponde ao ângulo esperado (atenção especial à confusão comum entre esquerda/direita do personagem vs esquerda/direita do observador) e se há consistência de proporções/cores.
  - Em personagens/criaturas rigáveis, verificar exposição clara das articulações (ombros, cotovelos, pelve, joelhos).
  - Se houver inconsistências graves que causariam falha na malha ou desperdício de créditos, pausar, apontar exatamente o problema encontrado e recomendar o ajuste antes de gastar créditos.

---

# 3. TRIPO — GERAÇÃO 3D, TOPOLOGIA E RIGGING (SDK OFICIAL V3)

### 3.0 Regra Estrita de Escopo (Proibição de Animações de Gameplay no Tripo)
Por padrão neste pipeline, o Tripo será utilizado **exclusivamente** para geração de geometria, texturização, otimização de malhas e rigging esquelético básico:
- Nunca utilizar endpoints de animação do Tripo para criar movimentos personalizados de gameplay sem pedido expresso do usuário.
- Nunca gerar ataques, magias, gestos de combate ou mecânicas específicas por prompt no Tripo.
- Nunca utilizar `One-Shot Animation` ou `Bake Animation` como substituto do sistema de animação do Blender ou da Game Engine alvo.
- Toda a lógica de animação de gameplay, blend trees e IK deverá ser implementada no Blender e/ou na Game Engine (ex.: Godot 4, Unity, Unreal).

### 3.1 Execução Assíncrona, Controle de Tarefas e Erros
- Utilizar `async/await` e `async with TripoClient()` como contexto de conexão dentro do `.venv` exclusivo do Tripo.
- Utilizar `wait_for_task()` e `download_task_models()`.
- A API pode retornar tarefas finalizadas com falha lógica sem lançar exceção Python. Após `wait_for_task()`, verificar obrigatoriamente:
  - `task.status` (confirmar explicitamente `TaskStatus.SUCCESS`).
  - `task.error_code` e `task.error_msg`.
  - `task.credits_consumed` e `task.output`. Caso `task.credits_consumed` seja nulo ou ausente (`None`), registrar o valor como `indisponível` e consultar `get_usage()` (nunca converter `None` em zero nem inventar um valor de consumo; quando presente, converter via `Decimal(str(task.credits_consumed))`).
  - Existência física, tamanho maior que zero e validade estrutural dos arquivos baixados no disco.
- Tratar separadamente as exceções do SDK: `TripoAPIError`, `TripoRequestError`, `TripoValidationError` e `TripoPollTimeoutError`.
- **Proteção contra cobrança dupla:** se houver timeout (`TripoPollTimeoutError`), queda de rede ou resultado desconhecido, persistir o `task_id` imediatamente após a criação da tarefa e consultá-lo através de `get_task(task_id)` antes de qualquer nova tentativa. **Nunca criar uma nova tarefa automaticamente** enquanto o status final da tarefa anterior não estiver esclarecido.

### 3.2 Seleção Explícita do Modelo de Geração, Single-view vs Multi-view e Formato P2
- Nunca depender silenciosamente do modelo padrão da API; definir explicitamente `model_version` em cada chamada (atualizando apenas caso a Tripo deprecie formalmente um identificador e registrando a mudança no relatório).
- **Modelo Padrão de Alta Fidelidade (H3.1):** `v3.1-20260211` para qualidade geral de forma e detalhes, confirmando compatibilidade com o modo escolhido (Single-view ou Multi-view). Não enviar `quad=True` nessa modalidade, pois a geração nativa em quads é exclusiva da linha P2. Quando forem necessários quads em um modelo H3.1, avaliar o processamento posterior por `smart_lowpoly(..., quad=True)`. Caso o H3.1 não seja suportado por determinada operação, selecionar uma versão oficialmente compatível e registrar a alteração.
- **Modelo Nativo Game-Ready (P2):** `P2-20260801`, com suporte nativo a `quad=True` (tanto em Single-view quanto em Multi-view, conforme documentação vigente), distinguindo estritamente seus parâmetros do H3.1:
  - Para geração *game-ready* direta em quads, utilizar `model_version="P2-20260801"` com os parâmetros oficialmente suportados, incluindo `quad=True` e `face_limit` dentro do intervalo oficial de `48` a `25000` faces quad.
  - `P2-20260801` **não suporta** os parâmetros `smart_low_poly` e `geometry_quality` na chamada de geração; nunca enviá-los nessa modalidade.
  - **Formato de saída P2:**
    - Quando `quad=True` é habilitado no P2, a API retorna a saída em `.fbx` para suportar faces quad.
    - Sempre confirmar a extensão real retornada pela URL do modelo e pelo resultado efetivo da tarefa (`.fbx`, `.glb` ou outra suportada), preservando a extensão verdadeira ao salvar no disco.
    - Caso seja `.fbx`, preservar a topologia quad editável ao importar no Blender.
    - Caso seja `.glb`, inspecionar sua geometria no Blender e confirmar se a malha veio triangulada.
    - Nunca renomear a extensão de um arquivo 3D sem conversão real via Blender.
  - Caso o usuário deseje benchmark de qualidade (P2 vs H3.1), comparar ambos usando exatamente a(s) mesma(s) imagem(ns) de referência e os mesmos parâmetros de textura (mediante autorização prévia do custo duplo).
  - Não aplicar automaticamente outra retopologia (`smart_lowpoly`) em um modelo P2 cuja malha quad já esteja limpa e dentro do orçamento de polígonos do jogo.
- **Escolha do Método Python conforme Quantidade de Imagens (1 vs 4):**
  - **Com 1 Imagem (Single-View):** utilizar `image_to_model(image=caminho_imagem, ...)`. Atenção à assinatura: no SDK Python utiliza-se o parâmetro singular `image`, enquanto o endpoint HTTP bruto utiliza `file`.
  - **Com 4 Imagens (Multi-View):** utilizar `multiview_to_model(images=[frontal, esquerda, costas, direita], ...)` respeitando a ordem obrigatória `[frontal, lateral_esquerda, traseira, lateral_direita]`, sendo a vista frontal indispensável. Atenção à assinatura: no SDK Python utiliza-se `images`, enquanto o endpoint HTTP bruto utiliza `inputs`. (Caso haja apenas 2 ou 3 vistas com a frontal presente, verificar a assinatura instalada do SDK para passagem de `None` nas posições ausentes).
- Registrar sempre nos logs e relatórios: modo (`image_to_model` vs `multiview_to_model`), quantidade de imagens, `model_version`, parâmetros enviados, formato esperado e extensão efetivamente baixada.

### 3.3 Controle de Topologia Quad e Smart Low Poly
- A geração inicial em H3.1 não possui `quad=True` nativo (entrega malha densa triangulada). Após baixar o modelo bruto (`_raw`), inspecionar sua topologia e contagem de faces no Blender antes de decidir pela retopologia ou rigging.
- Quando necessário reduzir polígonos ou obter topologia quad limpa a partir de um modelo H3.1 (ou refinar um prop denso), utilizar como referência:
  `smart_lowpoly(model_version="v3.0", face_limit=10000, quad=True)` *(ajustando `face_limit` ao orçamento do asset no jogo, ex.: 1.500–4.000 para props simples, 5.000–10.000 para personagens).*
- **Regras técnicas do Smart Low Poly:**
  - Validar a assinatura conforme a versão instalada do SDK (`model_version` do Smart Low Poly aceita versões próprias do processador como `v1.0`, `v2.0` ou `v3.0`; **não confundir** as versões de geração `P2-20260801` / `v3.1-20260211` com as versões do processador `smart_lowpoly`).
  - Com `quad=True`, respeitar estritamente o limite máximo documentado para essa operação (10.000 faces).
  - Preservar a extensão efetivamente retornada pela API (nunca presumir cegamente que toda saída quad será `.fbx` ou `.glb`; converter via Blender apenas quando necessário).
  - Executar sempre gerando um novo artefato (`_lowpoly`), preservando intacto o arquivo `_raw` original, mediante consulta de custo e autorização prévia do usuário.
  - Comparar visualmente e geometricamente o `_raw` e o `_lowpoly` no Blender (silhueta, preservação de UVs/texturas e loops em articulações) antes de escolher qual malha seguirá adiante.
- Não aplicar Smart Low Poly automaticamente se a malha original já atender ao orçamento e qualidade exigidos.

### 3.4 Matriz de Compatibilidade de Rigging, Retargeting e Especificação (`spec`)
> [!TIP]
> **Nota para Props e Cenários:** Se o asset for um objeto estático (arma rígida, móvel, item coletável, construção, vegetação sem esqueleto), **ignorar completamente esta seção (3.4) e as etapas de Rigging**, economizando créditos e seguindo direto para inspeção/pivô no Blender (Seção 4).

Para personagens e criaturas articuladas:
- Não utilizar cegamente uma única versão de rigging para todas as anatomias.
- Diferenciar capacidade de gerar esqueletos da disponibilidade de presets internos da Tripo:
  - A documentação oficial indica que `v2.5-20260210` e `v3.0-20260909` geram esqueletos para múltiplos tipos anatômicos, recomendando preferencialmente a linha `3.0` para humanoides (`biped`) e `2.5` para criaturas não humanoides.
  - Como as animações de gameplay serão criadas/retargetadas no Blender ou na Game Engine, priorizar a qualidade anatômica dos ossos e pesos (`skinning`), não os presets internos do Tripo.
- **Assinatura do método no SDK (`rig_model` vs `rig`):**
  - Conferir a assinatura real no SDK instalado e não misturar nomes de parâmetros: preferir `rig_model(..., model_version="...")` quando utilizar `rig_model`. Se utilizar o alias `rig()`, passar a versão pelo parâmetro `model` (`rig(..., model="...")`), conforme assinatura oficial instalada.
- **Seleção preferencial por anatomia:**
  - **Humanoides (`rig_type="biped"`):** priorizar `v3.0-20260909` com `spec="mixamo"`, quando compatível (facilita retargeting universal com `SkeletonProfileHumanoid` na Godot 4, `Humanoid Avatar` na Unity e `IK Retargeter` na Unreal).
  - **Aves (`avian`), quadrúpedes (`quadruped`), hexápodes, octópodes, serpentinos, aquáticos e outras criaturas:** utilizar o `rig_type` efetivamente detectado pelo Rig Check com `v2.5-20260210` como preferência inicial (utilizar `v3.0-20260909` em criaturas apenas se houver justificativa técnica testada).
- **Presets (`spec`):**
  - O parâmetro `spec` só se aplica a `rig_type="biped"` em combinações oficialmente suportadas.
  - Para não humanoides, omitir `spec` (ou manter o padrão nativo `tripo`). **Nunca forçar `spec="mixamo"` em criaturas não bípedes.**
- Executar previamente `rig_check()` / `check_riggable()` (verificando se há tarifação na versão atual da API) e analisar `output.riggable` e `output.rig_type`.
- **Nunca executar Auto Rig se `output.riggable` for `False`.** Informar o motivo ao usuário e avaliar ajustes na malha/pose no Blender se necessário.
- Registrar método chamado, versão de rigging, `rig_type`, `spec` e retorno completo do Rig Check.

### 3.5 Preservação das Versões (`raw`, `lowpoly` e `rigged`), Origem e Formato do Rig
- A versão `_raw` (sem rig e sem redução) baixada logo após a geração inicial nunca deve ser sobrescrita, mantendo sua extensão verdadeira (`.fbx` ou `.glb`, ex.: `heroi_raw.fbx`).
- Caso o Smart Low Poly seja aprovado, salvar separadamente (ex.: `heroi_lowpoly.fbx` ou `.glb`).
- **Origem para Rig Check e Auto Rig:**
  - Se a malha aprovada para rigging for a `_raw` original sem edições externas, utilizar diretamente seu `raw_task_id` como entrada.
  - **Fluxo para modelos `_lowpoly` (ou malhas editadas/corrigidas no Blender):**
    - As operações de Rig Check e Auto Rig na API atual aceitam diretamente um `file_token` obtido via upload (realizando a normalização internamente).
    - **Caminho principal (direto via `file_token`):** pegar o arquivo `_lowpoly` (ou exportado do Blender), enviá-lo via `upload_file()` para obter um `file_token` e passar esse `file_token` diretamente para o Rig Check e o Auto Rig (**nunca passar `lowpoly_task_id` diretamente** como se fosse um `original_model_task_id` de geração caso a API restrinja esse campo).
    - **Caminho alternativo (`import_model`):** utilizar `import_model(file_token)` -> `wait_for_task()` -> `import_task_id` somente quando a assinatura específica da versão instalada do SDK exigir estritamente um `task_id` e não aceitar `file_token` direto (verificando eventual custo e pedindo autorização antes).
- **Formato de saída no Auto Rig (preservação de Quads):**
  - A API de rigging suporta `glb` e `fbx` como saída (`out_format`).
  - Para malhas quad que precisam permanecer editáveis em quads no Blender, priorizar `out_format="fbx"` no Auto Rig.
  - Para malhas já trianguladas sem necessidade de edição em quads, permitir `out_format="glb"`.
  - Sempre verificar o formato realmente retornado pela API, salvando `<nome_do_asset>_rigged` com sua extensão verdadeira e deixando a triangulação final somente para o momento de exportar da cena do Blender para a Game Engine.
- Manter registrados todos os `task_id`, `file_token` e eventuais `import_task_id`.
- Se a geometria precisar ser alterada de forma significativa depois do rigging, avaliar se os pesos podem ser preservados/transferidos no Blender ou se será necessário refazer o rig.

### 3.6 Validação e Parâmetros Explícitos de Texturização
- **Atenção crítica ao parâmetro `delight`:** ao especificar `texture_version="v3.5-20260815"`, o padrão implícito da API para `delight` passa a ser `True`. Nunca deixar esse comportamento implícito — **definir sempre todos os parâmetros de textura explicitamente** de acordo com a direção de arte do jogo:
  - **Jogos Estilizados (Hand-Painted, Anime/Toon, Claymation, Low-Poly Artístico, Pixel-3D):** usar `delight=False` para preservar as cores ricas, gradientes pintados e identidade artística das referências 2D.
  - **Jogos PBR Realistas / Semi-Realistas com Iluminação Dinâmica Forte:** usar `delight=True` para remover sombras e reflexos "assados" da imagem 2D, evitando conflito com as luzes em tempo real da engine.
- **Parâmetros base recomendados para geração com texturas:**
  - `texture=True`
  - `pbr=True`
  - `texture_quality="standard"` *(ou `"detailed"` / `"HD"` quando suportado pelo SDK e autorizado pelo usuário para personagens principais)*
  - `texture_version="v3.5-20260815"`
  - `delight=False` *(ou `True`, conforme definido na Configuração do Projeto)*
- **Para testes exclusivamente geométricos / blocagem (desde que autorizado):**
  - `texture=False`
  - `pbr=False`
- Em eventuais comparações entre modelos (ex.: H3.1 vs P2), manter exatamente os mesmos parâmetros de textura.
- **Validação pós-geração no Blender:**
  - Verificar integridade de materiais Principled BSDF, resolução das texturas, UV Maps sem sobreposições destrutivas, canais PBR (Base Color, Roughness, Metallic, Normal) e ausência de texturas faltantes (cor magenta).
  - Se a textura vier ausente ou corrompida, avaliar `texture_model()` apenas como etapa corretiva opcional (mediante nova consulta de custo e autorização explícita). Nunca rodar `texture_model()` redundantemente se os materiais gerados já estiverem íntegros.

---

# 4. BLENDER MCP — INSPEÇÃO, CORREÇÃO E VALIDAÇÃO

### 4.1 Importação, Ambiente `bpy` e Diagnóstico
- Executar todos os scripts `bpy` exclusivamente dentro do ambiente Python do Blender (via Blender MCP ou Blender CLI).
- Importar o asset em uma `Collection` própria e organizada. Trabalhar sempre sobre cópia/arquivo `.blend` de trabalho (nunca sobrescrever os arquivos brutos `_raw`, `_lowpoly` ou `_rigged` baixados da Tripo).
- **Inspecionar geometria:** dimensões reais em metros (escala de mundo real compatível com a engine, ex.: humanoide ~1.70m–1.80m salvo proporção estilizada intencional), contagem de vértices/faces/triângulos, UV Maps, materiais, texturas embutidas e orientação das normais (faces invertidas).
- **Inspecionar esqueleto (quando rigado):** nomes dos ossos, hierarquia de parentesco, simetria L/R, `Bone Rest` e orientação de eixos locais.

### 4.2 Validação de Topologia (Crucial para Assets Gerados por IA)
- Não presumir que uma malha quad gerada por IA esteja automaticamente pronta para deformação.
- Em personagens e criaturas: inspecionar fluxo de arestas em ombros, cotovelos, pulsos, virilha/quadril, joelhos e pescoço, além de verificar vértices soltos, geometria interna fantasma ou *non-manifold* severo.
- Em props estáticos: verificar limpeza de silhueta, ausência de buracos na parte inferior/traseira (especialmente quando gerados com apenas 1 imagem!) e alinhamento correto do ponto de origem/pivô.

### 4.3 Pesos (`Skinning`) e Vertex Groups
- Não recalcular `Automatic Weights` automaticamente por cima de um rig funcional do Tripo (o cálculo automático cego do Blender frequentemente vaza pesos entre pernas, dedos ou roupas).
- Preservar os `Vertex Groups` originais caso as deformações estejam limpas.
- Se for necessário corrigir pesos: salvar backup, duplicar o objeto na cena, aplicar a correção/suavização de pesos na cópia, testar poses extremas e só então promover a cópia como principal.

### 4.4 Inspeção Visual Obrigatória (Screenshots / Previews)
- Para modelos rigados, ativar `armature_obj.show_in_front = True` para visualizar o alinhamento interno dos ossos sobre a malha.
- Capturar screenshots da viewport nas vistas **frontal, lateral e traseira** para inspecionar visualmente proporções, texturas e alinhamento de ossos.
- **Fallback automático:** se a captura direta de viewport falhar ou não existir no servidor Blender MCP instalado, renderizar previews diagnósticos rápidos (Workbench / EEVEE) com câmera ortográfica e iluminação neutra temporárias (removendo-as ao final sem sujar a cena) e salvar as imagens em `assets/importar/relatorios/` para inspeção visual do Claude.

### 4.5 Correções Anatômicas, Pivô, Testes de Deformação e Segurança de `Ctrl+A`
- **Para personagens e criaturas rigadas:**
  - Confirmar `Hips`/`Pelvis` centralizado na pelve real, ombros na articulação correta e cotovelos/joelhos com sutil indicação de ângulo natural para IK previsível.
  - Testar rotações básicas de pose (levantar e baixar braços, flexionar cotovelos/joelhos, girar cabeça) verificando esmagamentos (*candy-wrapper*) ou vértices presos na origem, **restaurando obrigatoriamente a Rest Pose (`pose.transforms_clear()`) após os testes**.
  - **Segurança de transformações (`Ctrl+A`):**
    - Nunca aplicar `Ctrl+A` (All Transforms) indiscriminadamente em objetos já rigados sem diagnóstico prévio.
    - Antes de qualquer aplicação de transformações em malha com Armature, inspecionar matrizes `matrix_world`, relação pai/filho, modificador Armature, `Bone Rest` e matrizes de *bind*. Qualquer normalização de escala/rotação deve ser feita em cópia de segurança e imediatamente validada com novos testes de pose.
  - Posicionar a sola dos pés tocando o plano do solo (`Z = 0`), movendo o conjunto raiz (`Armature` + `Mesh` vinculada) de modo a preservar as matrizes de vinculação.
- **Para props e objetos estáticos:**
  - Aplicar Rotação e Escala (`Scale = 1, 1, 1`) de forma limpa após alinhar o objeto aos eixos globais (`-Y` ou `+Z` frontal conforme padrão da engine alvo).
  - Posicionar a origem (pivô) de forma funcional para level design/gameplay: na base central inferior (`Z = 0`) para objetos apoiados no chão/mesas, no centro de massa para projéteis/física livre, ou na empunhadura para armas e itens equipáveis na mão.

---

# 5. EXPORTAÇÃO FINAL, PRESERVAÇÃO DE QUADS E ORGANIZAÇÃO

- **Papel de cada formato ao longo do pipeline:**
  - **Saída P2 quad / Modelo otimizado (`_raw` / `_lowpoly`):** `.fbx` quando retornado em quad (preservado intocado em `assets/importar/trabalho/tripo/`).
  - **Auto Rig de malha quad (`_rigged`):** `.fbx` (priorizando `out_format="fbx"` quando suportado) para manter os quads editáveis ao entrar no Blender.
  - **Arquivo mestre editável (`.blend`):** salvo em `assets/importar/trabalho/<nome_do_asset>.blend`, preservando geometria quad editável, materiais limpos e rig validado.
  - **Entrega final para Game Engine (`.glb` por padrão, ou `.fbx` se exigido pela engine):** exportado em `assets/modelos/<nome_do_asset>.glb`.
    - *Nota sobre glTF/GLB:* a especificação glTF 2.0 armazena malhas de renderização em triângulos. Portanto, manter quads no `.blend` para facilidade de edição futura e realizar a **triangulação controlada apenas na exportação final** pelo Blender (caso o projeto use Unreal/Unity com pipeline `.fbx` quad, exportar de acordo com a convenção do projeto).
- **Regras obrigatórias de exportação:**
  - Exportar apenas a coleção/objetos finais do asset (`Mesh` + `Armature` se houver), **excluindo câmeras, luzes temporárias, objetos de backup ocultos e poses de teste**.
  - Garantir que materiais PBR, texturas, UV Maps, normais customizadas e *skinning* (pesos de vértices) estejam incluídos e íntegros.
  - **Nunca sobrescrever** um arquivo existente em `assets/modelos/` sem antes copiar a versão anterior para `assets/importar/backups/` com timestamp.
  - **Validação pós-exportação (Round-Trip Check):** reimportar o `.glb` (ou `.fbx` final) recém-exportado em uma **cena limpa** do Blender via script de verificação para confirmar escala `1.0`, texturas carregadas, hierarquia de ossos e ausência de artefatos na triangulação antes de entregá-lo à Game Engine (não reprovar o `.glb` reimportado pelo fato esperado de estar triangulado).

---

# 6. INTEGRAÇÃO NA GAME ENGINE (GODOT 4 PADRÃO + NOTAS MULTI-ENGINE)

### 6.1 Verificação de Acesso e Validação Honesta na Engine
- Verificar primeiro se existe acesso real ao editor/CLI da Game Engine alvo ou servidor MCP específico (ex.: Godot CLI / Godot MCP, Unity MCP, Unreal Python).
- Se houver integração executável disponível, realizar a importação automatizada e os testes reais de cena, materiais e esqueleto.
- **Se não houver acesso direto à execução da Game Engine:**
  - Validar via sistema de arquivos tudo o que estiver acessível (ex.: `project.godot`, arquivos `.import`, estrutura de pastas).
  - Registrar honestamente a etapa no estado `VALIDACAO_ENGINE` como **pendente de validação visual no editor**.
  - Fornecer ao usuário um checklist objetivo e passo a passo (em Português do Brasil) do que clicar/conferir no editor da sua engine.
  - **Regra de integridade:** nunca declarar que `BoneMap`, `Humanoid Avatar`, `IK Rig`, colisores ou animações foram testados em runtime sem execução real comprovada.

### 6.2 Importação, Retargeting e Categorias de Asset
- **Na Godot 4 (Padrão):**
  - Modelos finais residem em `res://assets/modelos/`. Validar escala métrica, orientação (`-Z` forward na Godot), `Skeleton3D` e materiais PBR.
  - **Humanoides:** nas configurações de *Advanced Import* do `.glb`, mapear o `BoneMap` utilizando `SkeletonProfileHumanoid` para habilitar retargeting limpo e compartilhamento de bibliotecas de animação (`Idle`, `Walk`, `Run`, `Jump`, `Attack`, etc.).
  - **Aves (`avian`), quadrúpedes e criaturas não humanoides:** trabalhar diretamente com o `Skeleton3D` nativo exportado no `.glb`, criando/reaproveitando animações compatíveis no Blender ou na própria Godot sem forçar perfil humanoide.
  - **Props estáticos:** configurar geração de colisão adequada (`StaticBody3D` com primitivas simples ou colisor convexo otimizado, evitando *Trimesh* pesado em objetos pequenos/dinâmicos).
- **Em outras Engines (quando configurado para Unity ou Unreal Engine 5):**
  - **Unity:** para bípedes compatíveis (`mixamo`), configurar *Rig -> Animation Type: Humanoid* e validar o *Avatar* (todos os ossos verdes e em T-Pose); para criaturas, usar *Generic*; para props, configurar *Prefab* com *Mesh Collider (Convex)* ou primitivas e materiais URP/HDRP.
  - **Unreal Engine 5:** importar `.glb` (Interchange) ou `.fbx` com *Import Skeletal Mesh*, criar `IK Rig` e `IK Retargeter` caso deseje compartilhar animações do Mannequin, ou `Static Mesh` com colisão simplificada para props.

### 6.3 Boas Práticas de IK e Animação Procedural de Gameplay (Referência Godot 4)
Ao estruturar animações procedurais ou camadas de gameplay para personagens na **Godot 4.3+**, seguir a arquitetura moderna baseada em `SkeletonModifier3D`:
- **Olhar para alvos / inimigos / pontos de interesse (Cabeça e Pescoço):** utilizar `LookAtModifier3D` com limites angulares naturais.
- **Apontar armas, conjurar magias ou alcançar objetos com as mãos/braços:** utilizar `TwoBoneIK3D` configurado com `Target` e `Pole Target` (para controlar a direção do cotovelo/joelho). **Nunca usar `LookAtModifier3D` em uma cadeia inteira de braço**, pois isso causa torção severa (*twist*) na malha do antebraço e pulso.
- **Ações superiores enquanto o personagem se move (ex.: atacar, defender, segurar lanterna/item ou gesticular correndo):** utilizar `AnimationTree` com `Bone Mask` (filtrando tronco superior/braço desejado) ou controle procedural local, preservando a locomoção nas pernas.
- Validar sempre os eixos locais dos ossos, a `Bone Rest` e a ordem da pilha de processamento dos modificadores dentro do `Skeleton3D`.

---

# 7. AUTOMAÇÃO DO PIPELINE (ORDEM DEFINITIVA DE PROCESSAMENTO)

### 7.1 Sequência Obrigatória de Execução
1. **Inspeção Inicial e Checagem das Imagens (1 ou 4):**
   - Verificar `assets/importar/referencias/<nome_do_asset>/`. Se não houver imagens, **parar e avisar o usuário** que é obrigatório fornecer **1 imagem (Single-View)** ou **4 imagens (Multi-View: frente, esquerda, costas, direita)** de qualquer fonte artística/IA.
   - Se houver imagens, inspecioná-las visualmente quanto à qualidade, fundo, pose/articulações (se rigável) e consistência entre vistas.
   - Verificar `.venv` exclusivo do Tripo, versão exata de `tripo3d>=1.0.0,<2.0.0`, presença de `TRIPO_API_KEY` e disponibilidade do Blender MCP.
2. **Orçamento e Autorização de Geração:**
   - Consultar `get_balance()` via SDK (convertendo com `Decimal(str(valor))`), apresentar modo (1 imagem vs 4 imagens), `model_version`, parâmetros de textura (`delight` explícito) e custo estimado, e **aguardar autorização explícita do usuário**.
3. **Geração 3D Assíncrona:**
   - **Com 1 imagem:** executar `image_to_model(image=...)`.
   - **Com 4 imagens (ou múltiplas vistas com frontal):** executar `multiview_to_model(images=[frontal, esquerda, costas, direita])`.
   - Passar `model_version` explícito: H3.1 (`v3.1-20260211` sem `quad=True`) ou P2 (`P2-20260801` com `quad=True`, `face_limit` válido e sem `smart_low_poly`/`geometry_quality`), aplicando os parâmetros explícitos de texturização da Seção 3.6. Confirmar `TaskStatus.SUCCESS`.
4. **Download Seguro (`_raw`):**
   - Baixar e preservar `<nome_do_asset>_raw` em `assets/importar/trabalho/tripo/` inspecionando a URL/resultado e mantendo sua extensão verdadeira (`.fbx`, `.glb`, etc.), sem jamais sobrescrevê-lo depois.
5. **Diagnóstico Inicial no Blender:**
   - Importar `_raw` via Python interno do Blender MCP, verificando se a topologia veio em quads ou triângulos, contagem de polígonos, escala, UVs e integridade das texturas.
6. **Avaliação de Otimização (`Smart Low Poly`):**
   - Avaliar se há necessidade real de `smart_lowpoly(model_version="v3.0", face_limit=..., quad=True)` (nunca aplicar automaticamente se um modelo P2 ou malha original já estiver dentro do orçamento e bem estruturada).
7. **Execução Opcional de Low Poly (Sob Autorização):**
   - Se recomendado e autorizado pelo usuário após apresentação do custo, executar em cópia e baixar `<nome_do_asset>_lowpoly` preservando sua extensão real.
8. **Comparação e Seleção de Malha:**
   - Comparar `_raw` e `_lowpoly` no Blender e definir a malha aprovada. *(Nota: se o asset for um **prop/objeto estático**, pular diretamente os passos 9 a 12 de Rigging e ir para o passo 13).*
9. **Preparação de Origem para Rigging (Apenas Personagens e Criaturas):**
   - Se a malha aprovada for `_raw` sem edições externas, utilizar seu `raw_task_id`.
   - Se a malha aprovada for `_lowpoly` (ou malha ajustada no Blender), enviá-la via `upload_file()` para obter um `file_token` e utilizá-lo diretamente no Rig Check e Auto Rig (recorrendo a `import_model(file_token)` -> `import_task_id` apenas quando exigido pela versão instalada do SDK, mediante verificação de custo, e nunca passando `lowpoly_task_id` diretamente onde não suportado).
10. **Rig Check:**
    - Executar `rig_check()` / `check_riggable()` sobre a malha selecionada e validar `output.riggable` e `output.rig_type`. Se `riggable` for `False`, interromper o rigging e relatar ao usuário.
11. **Auto Rig (Sob Autorização):**
    - Consultar custo, aguardar autorização explícita do usuário e executar `rig_model(..., model_version="...")` (ou `rig(..., model="...")` conforme assinatura real):
      - Bípedes humanos (`biped`): priorizar `v3.0-20260909` com `spec="mixamo"`.
      - Criaturas (`avian`, `quadruped`, etc.): priorizar `v2.5-20260210` omitindo `spec`.
      - Priorizar `out_format="fbx"` para malhas quad editáveis (ou `"glb"` quando não houver necessidade de preservar quads).
12. **Download Seguro (`_rigged`):**
    - Baixar e preservar `<nome_do_asset>_rigged` separadamente em `assets/importar/trabalho/tripo/` com sua extensão verdadeira.
13. **Inspeção Visual e Validação no Blender:**
    - Validar geometria, materiais, pivô e (se rigado) esqueleto e pesos, gerando screenshots/previews frontais, laterais e traseiros (com `show_in_front = True` para armatures).
14. **Ajustes Seguros, Testes de Pose e Salvamento `.blend`:**
    - Aplicar correções seguras em cópia de trabalho (pés/base em `Z = 0`, pivô correto em props, testes de deformação em rigs restaurando a Rest Pose ao final) e salvar `assets/importar/trabalho/<nome_do_asset>.blend` preservando quads quando houver.
15. **Exportação Final e Round-Trip Check:**
    - Fazer backup prévio caso já exista versão anterior em `assets/modelos/`. Exportar o arquivo final (`.glb` triangulado na exportação, ou formato da engine) para `assets/modelos/` e reimportá-lo em cena limpa do Blender para confirmar integridade total.
16. **Integração na Game Engine:**
    - Integrar e validar na Game Engine alvo (ex.: Godot 4), ou registrar honestamente como pendente em `VALIDACAO_ENGINE` entregando o guia passo a passo ao usuário.
17. **Auditoria Financeira Final e Relatório:**
    - Consultar `get_usage()`, consolidar todos os `credits_consumed` via `Decimal(str(valor))` (ou indicar `indisponível` se `None`, nunca convertendo para zero), listar todos os IDs e `file_token` e salvar o relatório técnico em `assets/importar/relatorios/`.

### 7.2 Estados do Pipeline
`AGUARDANDO_REFERENCIAS` -> `REFERENCIAS_VALIDAS` -> `AGUARDANDO_AUTORIZACAO_TRIPO` -> `GERACAO_3D` -> `INSPECAO_GEOMETRIA` -> `OTIMIZACAO_LOWPOLY` *(opcional)* -> `IMPORTACAO_LOWPOLY` *(se aplicável)* -> `RIG_CHECK` *(apenas rigáveis)* -> `RIGGING` *(apenas rigáveis)* -> `IMPORTACAO_BLENDER` -> `DIAGNOSTICO` -> `CORRECAO` -> `VALIDACAO_RIG` *(ou `VALIDACAO_PROP`)* -> `EXPORTACAO` -> `VALIDACAO_ENGINE` -> `CONCLUIDO` | `FALHA`.

### 7.3 Relatórios Obrigatórios
Ao finalizar cada asset, gerar um relatório Markdown detalhado em `assets/importar/relatorios/<nome_do_asset>_relatorio.md` contendo:
- Nome e categoria do asset (Humanoide, Criatura ou Prop Estático) e Engine alvo;
- Modo de entrada utilizado (**Single-View 1 imagem** vs **Multi-View N imagens**) e caminhos das imagens;
- Versões exatas utilizadas: Python, SDK `tripo3d`, modelo de geração (`model_version`), textura (`texture_version`, `delight`, `pbr`, `texture_quality`), Smart Low Poly (se usado) e Auto Rig (se usado);
- Todos os `task_id`, `file_token` e eventuais `import_task_id` registrados;
- Consumo exato de créditos em cada etapa via `Decimal(str(valor))` (ou indicação explícita de `indisponível` + retorno de `get_usage()` caso `None`);
- Contagem de vértices, faces, triângulos e ossos (em quad no `.fbx`/`.blend` e triangulado no `.glb` final);
- Caminhos e formatos reais dos arquivos `_raw`, `_lowpoly`, `_rigged`, `.blend` e arquivo final em `assets/modelos/`;
- Problemas detectados na malha/textura/rig, correções aplicadas no Blender e screenshots gerados;
- Status real e honesto da validação na Game Engine.

---

# 8. PADRÃO DE QUALIDADE FINAL

Todos os assets entregues devem respeitar a direção artística definida para o jogo, apresentar silhueta limpa, escala métrica consistente, materiais PBR íntegros sem texturas quebradas, pivô corretamente posicionado, topologia/rig anatômico adequado à categoria do modelo e compatibilidade comprovada com a Game Engine alvo.

**REGRA DE OURO:** Automatizar o máximo possível para acelerar o desenvolvimento do jogo, mas **nunca sacrificar a direção artística, a precisão financeira dos créditos ou a integridade técnica do gameplay**. Jamais afirmar que um asset, rig ou integração na engine está concluído sem validação real.
