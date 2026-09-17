[README.pt-PT.md](https://github.com/user-attachments/files/32346333/README.pt-PT.md)[Uploading README# Workshop de extração de dados do YouTube

[English](README.md) | [Português (Portugal)](README.pt-PT.md)

Este repositório contém um notebook para Google Colab destinado à recolha de metadados de vídeos, comentários de primeiro nível e identificadores de utilizadores de um canal do YouTube através da YouTube Data API v3. Foi concebido para fins letivos e exercícios de investigação de pequena escala. Os limites predefinidos permitem concluir a extração durante um workshop.

## O que faz o notebook
[README.md](https://github.com/user-attachments/files/32346337/README.md)

O notebook:

1. identifica a lista de reprodução de carregamentos associada a um canal;
2. obtém os identificadores dos vídeos carregados pelo canal;
3. seleciona os vídeos pela data de publicação e duração mínima;
4. recolhe os comentários de primeiro nível publicados no período selecionado;
5. cria ficheiros CSV consolidados e separados por vídeo;
6. regista os resultados da extração, incluindo vídeos indisponíveis e comentários desativados; e
7. reúne os resultados num ficheiro ZIP.

As respostas aos comentários de primeiro nível não são recolhidas.

## Requisitos

- Uma conta Google
- Um projeto no Google Cloud com a YouTube Data API v3 ativada
- Uma chave da YouTube Data API
- Google Colab, recomendado para o workshop

O notebook instala `google-api-python-client` e `isodate`. O `pandas` já está disponível no Google Colab.

## Obter uma chave da API

1. Abra a [Google Cloud Console](https://console.cloud.google.com/).
2. Crie um projeto ou selecione um projeto existente.
3. Aceda a **APIs e serviços → Biblioteca**.
4. Ative a **YouTube Data API v3**.
5. Aceda a **APIs e serviços → Credenciais**.
6. Selecione **Criar credenciais → Chave de API**.

A chave da API não fica guardada no notebook. Quando a célula correspondente é executada, o Colab solicita-a através de um campo oculto. Nunca inclua uma chave de API num repositório público. Para utilizações posteriores ao workshop, restrinja a chave à YouTube Data API v3 e reveja as restrições de credenciais disponíveis.

## Executar o notebook

1. Descarregue ou clone este repositório.
2. Abra `youtube_data_extraction_workshop.ipynb` no Google Colab.
3. Edite a célula de configuração.
4. Execute as células pela ordem apresentada.
5. Introduza a chave da API quando esta for solicitada.
6. Consulte o registo da extração e descarregue o ficheiro ZIP.

## Configuração

Em condições normais, apenas é necessário alterar a célula de configuração.

| Variável | Finalidade | Predefinição do workshop |
| --- | --- | --- |
| `CHANNEL_ID` | ID do canal do YouTube iniciado por `UC` | ID do canal de Alan Barroso |
| `CHANNEL_NAME` | Nome utilizado para a pasta de resultados | `Alan_Barroso` |
| `START_DATE` | Primeira data incluída, interpretada em UTC | `2025-01-01` |
| `END_DATE` | Última data incluída, interpretada em UTC | `2025-12-31` |
| `MIN_VIDEO_DURATION_MINUTES` | Duração mínima dos vídeos | `8` |
| `MAX_VIDEOS` | Número máximo de vídeos que cumprem os critérios | `5` |
| `MAX_COMMENTS_PER_VIDEO` | Número máximo de comentários de primeiro nível por vídeo | `100` |
| `SAVE_TO_DRIVE` | Guardar os resultados no Google Drive | `True` |
| `OUTPUT_ROOT` | Pasta principal dos resultados | `youtube_workshop_data` |

Defina `MAX_VIDEOS` ou `MAX_COMMENTS_PER_VIDEO` como `None` para retirar o respetivo limite. Esta alteração pode aumentar consideravelmente o tempo de execução, o espaço ocupado e o consumo da quota da API.

## Estrutura dos resultados

Quando o armazenamento no Google Drive está ativado, os resultados são criados em:

```text
MyDrive/
└── youtube_workshop_data/
    └── CHANNEL_NAME/
        └── START_DATE_END_DATE/
            ├── videos.csv
            ├── comments.csv
            ├── users.csv
            ├── extraction_log.csv
            ├── comments_by_video/
            │   └── VIDEO_ID_comments.csv
            └── users_by_video/
                └── VIDEO_ID_users.csv
```

O notebook também cria um ficheiro ZIP com a pasta correspondente ao período selecionado.

### Ficheiros principais

- `videos.csv`: ID, título, data e hora de publicação, duração, número de visualizações e número de comentários indicado pelo YouTube.
- `comments.csv`: uma linha por comentário de primeiro nível, com o vídeo, dados do autor, texto, datas, gostos e número de respostas.
- `users.csv`: pares distintos de ID do canal do autor e nome apresentado encontrados nos comentários recolhidos.
- `extraction_log.csv`: uma linha por vídeo selecionado, com o estado da extração e o número de comentários recolhidos.

O nome apresentado pelo autor não é um identificador estável. Quando o YouTube o disponibiliza, o notebook conserva o `author_channel_id`; ainda assim, este valor pode estar ausente em alguns comentários.

## Funcionamento da extração

- Os detalhes dos vídeos são solicitados em lotes de até 50 IDs.
- Cada página pode conter até 100 tópicos de comentários; as páginas seguintes são obtidas através de `nextPageToken`.
- Os comentários são ordenados por data, permitindo terminar a extração quando se alcança o início do período selecionado.
- Os erros temporários do servidor originam novas tentativas com intervalos progressivos.
- A extração termina explicitamente quando a quota se esgota.
- Comentários desativados, vídeos indisponíveis ou inexistentes, períodos sem comentários e limites do workshop ficam registados no ficheiro de extração.

## Âmbito e limitações

O acesso através da API não torna o YouTube inteiramente observável. Conteúdos eliminados, privados, restringidos ou indisponíveis por outros motivos podem não ser recuperados. As contagens e a disponibilidade podem mudar depois da recolha; por isso, a data da extração e a configuração utilizada devem acompanhar qualquer análise.

O período de amostragem aplica-se tanto às datas de publicação dos vídeos como às datas de publicação dos comentários. Cabe ao investigador definir a amostra, o limiar de duração, as regras de exclusão e a unidade de análise.

## Utilização responsável

O caráter público dos dados não dispensa a minimização dos dados, o armazenamento seguro, a avaliação ética e o tratamento cuidadoso dos identificadores dos utilizadores. Antes de recolher ou partilhar dados, devem ser considerados os requisitos institucionais, o enquadramento jurídico, os termos da plataforma e a finalidade da investigação.

## Resolução de problemas

| Mensagem ou estado | Causa provável | Ação sugerida |
| --- | --- | --- |
| `No API key was provided` | O campo da chave ficou vazio | Execute novamente a célula e introduza uma chave válida |
| `The channel was not found` | O ID do canal está incorreto | Utilize o ID iniciado por `UC`, não o identificador precedido por `@` nem o URL do canal |
| `No videos matched...` | Nenhum vídeo cumpre os filtros de data e duração | Reveja as datas e `MIN_VIDEO_DURATION_MINUTES` |
| `comments_disabled` | Os comentários estão desativados nesse vídeo | Não é possível recolher comentários através deste método |
| `quotaExceeded` ou `dailyLimitExceeded` | A quota diária do projeto foi esgotada | Aguarde pela reposição da quota ou reveja as definições de quota do projeto |

# YouTube Data Extraction Workshop

[English](README.md) | [Português (Portugal)](README.pt-PT.md)

This repository contains a Google Colab notebook for collecting video metadata, top-level comments, and user identifiers from a YouTube channel through the YouTube Data API v3. It is intended for teaching and small-scale research exercises. The default limits keep the extraction short enough for a workshop.

## What the notebook does

The notebook:

1. identifies the uploads playlist associated with a channel;
2. retrieves the channel's uploaded video identifiers;
3. selects videos by publication date and minimum duration;
4. collects top-level comments published within the selected period;
5. creates consolidated and per-video CSV files;
6. records extraction outcomes, including unavailable videos and disabled comments; and
7. packages the results in a ZIP archive.

Replies to top-level comments are not collected.

## Requirements

- A Google account
- A Google Cloud project with the YouTube Data API v3 enabled
- A YouTube Data API key
- Google Colab, recommended for the workshop

The notebook installs `google-api-python-client` and `isodate`. `pandas` is already available in Google Colab.

## Obtain an API key

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a project or select an existing one.
3. Go to **APIs & Services → Library**.
4. Enable **YouTube Data API v3**.
5. Go to **APIs & Services → Credentials**.
6. Select **Create credentials → API key**.

The API key is not stored in the notebook. When the relevant cell runs, Colab requests it through a hidden input field. Never commit an API key to a public repository. For use beyond the workshop, restrict the key to the YouTube Data API v3 and review the available credential restrictions.

## Run the notebook

1. Download or clone this repository.
2. Open `youtube_data_extraction_workshop.ipynb` in Google Colab.
3. Edit the configuration cell.
4. Run the cells in order.
5. Enter the API key when prompted.
6. Review the extraction log and download the ZIP archive.

## Configuration

Only the configuration cell normally needs to be changed.

| Variable | Purpose | Workshop default |
| --- | --- | --- |
| `CHANNEL_ID` | YouTube channel ID beginning with `UC` | Alan Barroso channel ID |
| `CHANNEL_NAME` | Name used for the output folder | `Alan_Barroso` |
| `START_DATE` | First date included, interpreted in UTC | `2025-01-01` |
| `END_DATE` | Last date included, interpreted in UTC | `2025-12-31` |
| `MIN_VIDEO_DURATION_MINUTES` | Minimum video duration | `8` |
| `MAX_VIDEOS` | Maximum number of matching videos | `5` |
| `MAX_COMMENTS_PER_VIDEO` | Maximum top-level comments per video | `100` |
| `SAVE_TO_DRIVE` | Save results in Google Drive | `True` |
| `OUTPUT_ROOT` | Root output folder | `youtube_workshop_data` |

Set `MAX_VIDEOS` or `MAX_COMMENTS_PER_VIDEO` to `None` to remove the corresponding limit. Doing so may substantially increase runtime, storage use, and API-quota consumption.

## Output structure

When Google Drive storage is enabled, results are created under:

```text
MyDrive/
└── youtube_workshop_data/
    └── CHANNEL_NAME/
        └── START_DATE_END_DATE/
            ├── videos.csv
            ├── comments.csv
            ├── users.csv
            ├── extraction_log.csv
            ├── comments_by_video/
            │   └── VIDEO_ID_comments.csv
            └── users_by_video/
                └── VIDEO_ID_users.csv
```

The notebook also creates a ZIP archive of the period folder.

### Main files

- `videos.csv`: video ID, title, publication time, duration, view count, and the comment count reported by YouTube.
- `comments.csv`: one row per top-level comment, including the video, author information, text, dates, likes, and reply count.
- `users.csv`: distinct pairs of author channel ID and displayed author name found in the extracted comments.
- `extraction_log.csv`: one row per selected video with its extraction status and number of comments collected.

The displayed author name is not a stable identifier. When YouTube provides it, the notebook retains `author_channel_id`; this value may still be absent for some comments.

## Extraction behaviour

- Video details are requested in batches of up to 50 IDs.
- Comment pages contain up to 100 threads and are followed through `nextPageToken`.
- Comments are ordered by time so that extraction can stop after reaching the beginning of the selected period.
- Temporary server errors are retried with exponential backoff.
- Quota exhaustion stops the extraction explicitly.
- Disabled comments, unavailable videos, missing videos, empty periods, and classroom limits are recorded in the extraction log.

## Scope and limitations

API access does not make YouTube fully observable. Deleted, private, restricted, or otherwise unavailable content may be absent. Counts and availability can change after collection, so the extraction date and configuration should be documented with any analysis.

The sampling period applies both to video publication dates and to comment publication dates. The researcher remains responsible for defining the sample, duration threshold, exclusion rules, and unit of analysis.

## Responsible use

Public availability does not remove the need for data minimisation, secure storage, ethical review, and careful handling of user identifiers. Before collecting or sharing data, consider the applicable institutional requirements, legal framework, platform terms, and research purpose.

## Troubleshooting

| Message or status | Likely cause | Suggested action |
| --- | --- | --- |
| `No API key was provided` | The prompt was left empty | Run the API-key cell again and enter a valid key |
| `The channel was not found` | The channel ID is incorrect | Use the channel ID beginning with `UC`, not a handle or channel URL |
| `No videos matched...` | No video meets the date and duration filters | Review the dates and `MIN_VIDEO_DURATION_MINUTES` |
| `comments_disabled` | Comments are disabled for the video | No comment extraction is possible through this method |
| `quotaExceeded` or `dailyLimitExceeded` | The project's daily quota has been exhausted | Wait for the quota to reset or review the project's quota settings |

.pt-PT.md…]()
