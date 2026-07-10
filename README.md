# 💀 DBD Build Manager

Aplicativo desktop em **Python + CustomTkinter** para gerenciar builds do **Dead by Daylight**, com banco de dados **SQLite** local, tema claro/escuro, backups automáticos, exportação em JSON/PNG e sistema de importação dos dados oficiais do jogo.

## Instalação

Requisitos: **Python 3.10+**

```bash
# 1. (opcional, recomendado) criar um ambiente virtual
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/macOS:
source .venv/bin/activate

# 2. instalar as dependências
pip install -r requirements.txt

# 3. executar
python main.py
```

Na primeira execução o app cria automaticamente o banco (`data/dbd_builds.db`), as pastas de ícones e um backup inicial.

## Estrutura do projeto (arquitetura MVC)

```
dbd_build_manager/
├── main.py                     # Ponto de entrada
├── config.py                   # Caminhos, constantes e settings
├── requirements.txt
├── database/
│   ├── db_manager.py           # Conexão e helpers do SQLite
│   └── schema.sql              # Schema completo (tabelas + índices)
├── models/                     # MODEL — acesso a dados
│   ├── game_data_repository.py # killers, perks, itens, add-ons
│   └── build_repository.py     # builds, tags, favoritos, estatísticas
├── controllers/                # CONTROLLER — orquestração
│   └── app_controller.py
├── services/                   # Serviços de apoio
│   ├── backup_service.py       # backup automático do banco
│   ├── data_importer.py        # importação JSON/API dos dados do jogo
│   ├── build_io_service.py     # exportar/importar builds em JSON
│   └── image_export_service.py # exportar build em PNG
├── views/                      # VIEW — interface (CustomTkinter)
│   ├── main_window.py          # janela principal + abas
│   ├── base_build_tab.py       # lista (esq.) + detalhes (dir.)
│   ├── survivor_tab.py         # aba Survivors
│   ├── killer_tab.py           # aba Killers
│   ├── build_editor.py         # criar/editar builds
│   ├── data_manager_window.py  # gerenciar dados do jogo
│   └── widgets/
│       ├── tooltip.py          # tooltips com descrição completa
│       ├── icon_cache.py       # cache de ícones + placeholder
│       └── picker_dialog.py    # seletor com busca rápida
├── data/
│   ├── dbd_builds.db           # banco SQLite (criado na 1ª execução)
│   ├── settings.json           # preferências (tema etc.)
│   ├── templates/              # modelos JSON para importar dados do jogo
│   └── icons/                  # ícones organizados por tipo
│       ├── killers/  killer_perks/  killer_addons/
│       └── survivor_perks/  survivor_items/  survivor_addons/
└── backups/                    # backups automáticos (mantém os 10 últimos)
```

## Importando os dados oficiais do jogo

O app é entregue **sem dados fictícios**: você importa o conteúdo real do DBD (e reimporta a cada atualização do jogo) de duas formas:

### 1. Arquivos JSON

Preencha os modelos em `data/templates/` com os dados oficiais (nomes, descrições completas e nomes dos arquivos de ícone) e coloque as imagens nas subpastas correspondentes de `data/icons/`. Depois:

**⚙ Dados do jogo → Importar JSON / API → Importar arquivo** (ou **Importar pasta** para todos de uma vez).

Ordem recomendada (por dependência): `killers.json` → demais arquivos. A importação faz **upsert por nome**: reimportar um JSON atualizado apenas adiciona/atualiza registros, sem duplicar — o sistema acompanha atualizações do DBD sem reescrever nada.

Formato de cada arquivo: veja o campo `_schema` dentro de cada template e os comentários em `services/data_importer.py`.

### 2. API

Se você tiver uma URL que retorne JSON no mesmo formato dos templates (por exemplo, um endpoint próprio ou um serviço da comunidade que você adapte), informe-a em **Dados do jogo → Importar da API**.

### Cadastro manual

Para adições pontuais (novo capítulo do jogo, por exemplo), a janela **⚙ Dados do jogo** tem formulários para cadastrar Killer, Perks, Itens e Add-ons individualmente, com seleção do arquivo de ícone.

## Funcionalidades

| Recurso | Onde |
|---|---|
| Criar / editar / duplicar / excluir builds | Botões da aba |
| Favoritar | Botão ★ (também usado na ordenação) |
| Busca rápida pelo nome | Campo 🔍 no painel esquerdo |
| Filtrar por perk (ambas as abas) e por Killer | Botões de filtro |
| Ordenação | Favoritos, criação, modificação, nome, mais usadas |
| Tooltips com descrição completa | Passe o mouse sobre perks, add-ons, itens e killers |
| Tags personalizadas | Campo no editor de build |
| Comentários + anotações estratégicas | Campos separados no editor |
| Estatísticas de uso | Botão **▶ Marcar uso** (contador + último uso) |
| Exportar build em PNG | Botão **🖼 PNG** |
| Exportar/importar builds em JSON | **⇩ JSON** (uma), **Exportar tudo / Importar builds** (barra superior) |
| Backup automático | A cada abertura do app; gerencie em **🗄 Backups** |
| Modo claro/escuro | Switch na barra superior (preferência é salva) |

## Escalabilidade / futuras atualizações do jogo

* **Schema normalizado**: builds referenciam perks/itens/add-ons por ID; os dados do jogo existem uma única vez.
* **Upsert por nome** em toda a importação: novos capítulos entram só com um novo JSON.
* **Camadas separadas (MVC)**: trocar a GUI, adicionar novas entidades (ex.: ofertas/offerings) ou uma fonte de dados nova exige mexer em apenas uma camada.
* **Builds exportadas por nome** (não por ID), portáteis entre instalações.

## Observações

* Sem ícones no disco, o app gera placeholders com as iniciais — tudo funciona antes mesmo de adicionar as imagens.
* Ao restaurar um backup (copiando o arquivo de `backups/` sobre `data/dbd_builds.db`), reinicie o aplicativo.

## Preenchendo com os dados oficiais do jogo (automático) ⭐

O jeito mais fácil — requer internet:

1. Abra o aplicativo (`python main.py`).
2. Clique em **⚙ Dados do jogo** no topo.
3. Na aba **Importar JSON / API**, clique em **⬇ Baixar dados oficiais da internet (automático)**.
4. Aguarde (baixa killers, perks, itens, add-ons, descrições e centenas de ícones — alguns minutos na primeira vez).

Alternativa por linha de comando (sem abrir o app): `python baixar_dados.py`

Fonte dos dados: API comunitária [dbd.tricky.lol](https://dbd.tricky.lol), que se mantém sincronizada com o jogo.
Saiu atualização do DBD? Clique no botão de novo — a importação atualiza pelo nome, sem duplicar nada.
Os dados baixados também ficam salvos em `data/templates/*.json` para reimportação offline.

## Gerando um .exe (abrir com dois cliques, sem Python)

1. Dê dois cliques em **gerar_exe.bat** (só na primeira vez / após atualizações).
2. Aguarde alguns minutos; ao final, abra a pasta **dist\\DBD Build Manager**.
3. Dê dois cliques em **DBD Build Manager.exe**. Crie um atalho na área de trabalho se quiser.

O banco de dados, ícones e backups ficam dentro dessa mesma pasta — para
fazer backup do app inteiro, basta copiar a pasta. Os dados já baixados
podem ser aproveitados copiando as pastas `data` e `backups` antigas para lá.

## Atualização automática (compartilhar com amigos)

O app confere sozinho, ao abrir, se existe versão nova num endereço da
internet — e oferece baixar e aplicar (preservando builds e dados).

### Configurar uma vez (você, o "dono")
1. Crie uma conta gratuita em github.com e um repositório **público**
   (ex.: `dbd-build-manager`).
2. No repositório, crie o arquivo `manifest.json` (botão *Add file →
   Create new file*), com o conteúdo gerado pelo `publicar_atualizacao.bat`.
3. O endereço do seu manifesto será:
   `https://raw.githubusercontent.com/SEU_USUARIO/SEU_REPOSITORIO/main/manifest.json`
4. Crie na pasta do aplicativo um arquivo `update_url.txt` contendo só
   essa linha (esse arquivo vai junto quando você distribuir o ZIP para
   os amigos — é ele que liga a atualização automática).

### Publicar cada nova versão
1. Aumente `APP_VERSION` em `config.py` (ex.: "1.2.0").
2. Dois cliques em **publicar_atualizacao.bat** → ele gera
   `publicar\DBDBuildManager-vX.Y.Z.zip` e um `manifest.json` modelo.
3. No GitHub: *Releases → Create a new release* → tag `vX.Y.Z` → anexe o
   ZIP → *Publish*. Copie o link do ZIP anexado.
4. Edite o `manifest.json` do repositório: versão nova, `zip_url` com o
   link copiado e as novidades em `notes`.
5. Pronto: na próxima vez que cada amigo abrir o app, ele avisa e
   atualiza sozinho (ou pelo botão **⟳ Atualizar**).

O que a atualização NUNCA toca: `data/` (builds, ícones, banco),
`backups/` e o `update_url.txt` de cada pessoa.

## Biblioteca de Builds da Comunidade

Compartilhe e descubra builds com outras pessoas que usam o app, direto
pelo mesmo repositório do GitHub já configurado para as atualizações
(sem criar conta nem configurar nada a mais).

**Para importar builds da galera:** clique em **🌐 Comunidade** no topo
do app. A lista carrega sozinha; clique em **⬇ Importar** na build que
quiser.

**Para compartilhar uma build sua:**
1. Exporte a build pelo botão **JSON** (aba Survivors ou Killers).
2. Dê um nome descritivo ao arquivo (ex.: `GenRush_TheGhoul_por_Voce.json`).
3. Na janela **🌐 Comunidade**, clique em **📤 Como compartilhar sua build**
   e depois em **🔗 Abrir página de envio no GitHub** — isso abre a pasta
   `builds/` do repositório pronta para você arrastar o arquivo e
   confirmar (Commit changes).
4. Pronto — na próxima vez que alguém abrir a Biblioteca, sua build
   já aparece na lista.

Requer que a atualização automática já esteja configurada (arquivo
`update_url.txt`) — veja a seção anterior deste README.
