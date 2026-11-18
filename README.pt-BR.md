# Pipeline 3D Universal para Jogos — Claude + Tripo SDK (v3) + Blender MCP

> 🇺🇸 **English:** [README.md](./README.md) | 🇧🇷 **Português:** `README.pt-BR.md`

Um pipeline técnico completo ([`CLAUDE.pt-BR.md`](./CLAUDE.pt-BR.md)) projetado para transformar o **Claude** (Claude Code / Claude Desktop) em um **Técnico de Produção 3D** capaz de gerar, retopologizar em *quads*, aplicar *Auto-Rig*, inspecionar/corrigir no **Blender** via MCP e exportar assets `.glb` / `.fbx` prontos para **Godot 4**, **Unity** ou **Unreal Engine**.

---

## Principais Diferenciais

- **Funciona para Qualquer Jogo e Estilo de Arte:** Estilizado (*hand-painted*, *claymation*, *toon/anime*, *low-poly*) ou PBR realista, com controle explícito de `delight` (`True` ou `False`).
- **Entrada Flexível (1 ou 4 Imagens de Qualquer Fonte):**
  - **1 Imagem (`Single-View`):** rápido e prático para *props*, itens e objetos de cenário (`image_to_model`).
  - **4 Imagens (`Multi-View` — Frente, Esquerda, Costas, Direita):** alta fidelidade anatômica para personagens e criaturas (`multiview_to_model`).
  - **Sem dependência de ferramenta 2D:** use artes próprias feitas à mão (Photoshop, Krita, Procreate, Blender Grease Pencil) ou qualquer IA geradora de imagens (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion / ComfyUI, Leonardo AI, etc.).
- **Segurança Estrita de Créditos (`Decimal`):** O agente consulta saldo antes, pede **autorização explícita** antes de cada etapa paga na API da Tripo, protege contra cobrança dupla em caso de *timeout* (`get_task`) e contabiliza créditos com `Decimal` (sem erros de ponto flutuante).
- **Preservação Real de Topologia Quad:** Suporte às regras específicas do modelo `P2` (`quad=True` em `.fbx`) e `Smart Low Poly`, mantendo o `.blend` editável em *quads* e triangulando apenas na exportação final do `.glb`.
- **Rigging Inteligente por Anatomia:** Seleção automática entre `v3.0` (`spec="mixamo"` para humanoides) e `v2.5` (para aves, quadrúpedes e monstros), além de pular rigging automaticamente em *props* estáticos para economizar créditos.
- **Inspeção Visual e Round-Trip no Blender MCP:** Validação de UVs, normais, alinhamento de ossos (`show_in_front = True`), testes de deformação e reimportação do `.glb` final em cena limpa antes de entregar para a engine.

---

## Pré-requisitos

1. **Python 3.10+** (para o ambiente isolado do SDK da Tripo)
2. **Conta e API Key da [Tripo 3D](https://platform.tripo3d.ai/)** configurada na variável de ambiente `TRIPO_API_KEY`
3. **Blender 3.6 / 4.x** + **[Blender MCP](https://github.com/ahujasid/blender-mcp)** configurado no seu cliente Claude
4. **Claude Code** ou **Claude Desktop**
5. *(Opcional)* **Godot 4**, **Unity** ou **Unreal Engine 5**

---

## Como Configurar no Seu Jogo

### 1. Copie o `CLAUDE.pt-BR.md` para a raiz do seu projeto como `CLAUDE.md`
Coloque o arquivo [`CLAUDE.pt-BR.md`](./CLAUDE.pt-BR.md) na pasta raiz do seu jogo (renomeando-o para `CLAUDE.md` para que o Claude Code o carregue automaticamente) e preencha (se desejar) os campos iniciais da seção **Configuração do Projeto** (nome, engine alvo e estilo visual).

### 2. Crie o ambiente virtual exclusivo do Tripo SDK
Na raiz do seu projeto de jogo, crie o `.venv` isolado exatamente onde o pipeline espera:

**Windows (PowerShell):**
```powershell
python -m venv tools/pipeline_3d/tripo/.venv
.\tools\pipeline_3d\tripo\.venv\Scripts\pip.exe install "tripo3d>=1.0.0,<2.0.0"
```

**Linux / macOS:**
```bash
python3 -m venv tools/pipeline_3d/tripo/.venv
./tools/pipeline_3d/tripo/.venv/bin/pip install "tripo3d>=1.0.0,<2.0.0"
```

### 3. Configure sua `TRIPO_API_KEY` no sistema
Nunca cole sua chave diretamente no chat ou no código! Defina como variável de ambiente:

**Windows (PowerShell — permanente para o usuário):**
```powershell
[Environment]::SetEnvironmentVariable("TRIPO_API_KEY", "tsk_sua_chave_aqui", "User")
```

**Linux / macOS (`~/.bashrc` ou `~/.zshrc`):**
```bash
export TRIPO_API_KEY="tsk_sua_chave_aqui"
```

---

## Passo Obrigatório Antes de Gerar: Coloque 1 ou 4 Imagens!

Antes de pedir ao Claude para gerar qualquer modelo 3D, crie uma pasta com o nome do asset dentro de `assets/importar/referencias/<nome_do_asset>/` e coloque:

- **Opção A — 1 Imagem (Single-View):**
  - Ex.: `assets/importar/referencias/bau_madeira/frente.png`
  - Ideal para *props*, itens simples ou objetos simétricos.
- **Opção B — 4 Imagens (Multi-View — Recomendado para Personagens/Criaturas):**
  - `assets/importar/referencias/guerreiro/frente.png`
  - `assets/importar/referencias/guerreiro/esquerda.png`
  - `assets/importar/referencias/guerreiro/costas.png`
  - `assets/importar/referencias/guerreiro/direita.png`
  - Para humanoides que receberão Auto-Rig, prefira **T-Pose** com fundo neutro/limpo e articulações de ombros e cintura/pelve visíveis.

> [!NOTE]
> Se você esquecer de colocar as imagens, o próprio Claude irá pausar na etapa inicial (`AGUARDANDO_REFERENCIAS`) e avisar onde adicioná-las antes de gastar qualquer crédito.

---

## Exemplo de Prompt para Iniciar no Claude

Com o Blender aberto (e o addon do Blender MCP ativo), basta pedir ao Claude:

> *"Siga o `CLAUDE.md` e processe o personagem em `assets/importar/referencias/guerreiro/` usando o modelo P2 em quads para a Godot 4."*

Ou para um objeto estático de cenário com apenas 1 imagem:

> *"Siga o `CLAUDE.md` e gere o prop estático que está em `assets/importar/referencias/bau_madeira/` (1 imagem, sem rig)."*

---

## Licença

Distribuído sob licença MIT. Sinta-se livre para usar, adaptar e contribuir!
