# UltraStar Web Edition

Um projeto para recriar o famoso jogo de karaokê UltraStar em tecnologias web modernas, mantendo todas as funcionalidades e a experiência de jogo do UltraStar original.

![UltraStar Web Edition](songs/Pearl%20Jam%20-%20Jeremy/Pearl%20Jam%20-%20Jeremy.jpg)

## Visão Geral

UltraStar Web Edition é uma reimplementação do UltraStar WorldParty em ambiente web. O objetivo é fornecer uma experiência de karaokê completa, acessível através do navegador, mantendo todas as funcionalidades do jogo original.

### Características Principais

- **Visualização de Notas**: Exibe notas musicais em uma linha do tempo rolante
- **Detecção de Tom**: Analisa o tom da voz do jogador em tempo real
- **Sistema de Pontuação**: Pontua com base na precisão do tom e timing
- **Efeitos Visuais**: Oferece feedback visual sobre a performance do jogador
- **Múltiplos Jogadores**: Suporta jogadores múltiplos (solo, duetos, competição)
- **Biblioteca de Músicas**: Gerencia um catálogo de músicas com letras, vídeos e áudio

## Tecnologias

### Frontend
- **Framework**: React.js para gerenciamento de componentes
- **Renderização**: HTML5 Canvas para desenho das letras e notas
- **Áudio**: Web Audio API
- **Detecção de Tom**: Algoritmo de autocorrelação personalizado ou biblioteca Pitchy.js

### Backend (opcional para versão multiplayer)
- **API**: Node.js + Express
- **Banco de Dados**: MongoDB para armazenar músicas, letras e pontuações
- **Armazenamento**: Possível integração com armazenamento em nuvem para arquivos de mídia

## Arquitetura

```
UltraStarWeb
│
├── Core
│   ├── GameEngine      - Controle principal do jogo
│   ├── BeatSystem      - Sistema de sincronização por beats
│   ├── SongManager     - Gerenciamento de músicas e arquivos
│   └── PlayerManager   - Gerenciamento de jogadores
│
├── Audio
│   ├── AudioEngine     - Reprodução de música
│   ├── PitchDetector   - Detecção de tom do microfone
│   └── AudioAnalyzer   - Análise de áudio para visualizações
│
├── Graphics
│   ├── LyricsRenderer  - Renderização de letras
│   ├── NoteRenderer    - Renderização de notas musicais
│   ├── EffectsManager  - Efeitos visuais das letras
│   └── VideoPlayer     - Reprodução de vídeos de fundo
│
└── UI
    ├── SongSelect      - Seleção de músicas
    ├── SingScreen      - Tela principal de jogo
    ├── ScoreScreen     - Tela de pontuação
    └── OptionsMenu     - Configurações do jogo
```

## Formato de Arquivo e Parsing

O UltraStar utiliza arquivos de texto (.txt) para definir as letras e notas:

```
#TITLE:Nome da Música
#ARTIST:Nome do Artista
#MP3:arquivo_de_musica.mp3
#VIDEO:arquivo_de_video.mp4
#BPM:120
#GAP:300
#VIDEOGAP:0

: 0 5 40 Primeira
: 5 5 36 parte 
: 10 5 38 da
: 15 10 40 letra
```

Cada linha começa com um prefixo que indica seu tipo:
- `:` - Nota normal
- `*` - Nota freestyle (não avaliada por tom)
- `F` - Nota dourada (pontuação dupla)
- `-` - Fim de linha/pausa

## Roadmap de Desenvolvimento

### Fase 1: Fundação (2 semanas)
1. Configurar projeto React e estrutura de pastas
2. Implementar parser de arquivos UltraStar
3. Criar sistema básico de beats e timing
4. Implementar reprodução básica de áudio

### Fase 2: Core do Jogo (3 semanas)
1. Desenvolver sistema de renderização de letras
2. Implementar detecção básica de pitch
3. Criar sistema de notas e avaliação
4. Implementar sistema de pontuação

### Fase 3: Experiência de Jogo (2 semanas)
1. Adicionar efeitos visuais para letras
2. Implementar feedback visual de desempenho
3. Criar interface de seleção de músicas
4. Adicionar tela de pontuação

### Fase 4: Polimento (2 semanas)
1. Otimizar desempenho de renderização
2. Refinar algoritmo de detecção de tom
3. Adicionar suporte para diferentes níveis de dificuldade
4. Implementar carregamento dinâmico de músicas

### Fase 5: Recursos Avançados (3 semanas)
1. Adicionar suporte para vídeos de fundo
2. Implementar modo de jogo para múltiplos jogadores
3. Criar sistema de playlists
4. Adicionar estatísticas e sistema de recordes

## Como Começar

### Configuração Inicial

```bash
# Criar aplicação React
npx create-react-app ultrastar-web

# Instalar dependências úteis
npm install howler pitchy webfontloader

# Iniciar servidor de desenvolvimento
npm start
```

### Estrutura de Arquivos

```
ultrastar-web/
├── public/
│   ├── songs/           
│       ├── # Arquivos de vídeos (.mp4)
│       ├── # Arquivos de letras (.txt)
|       ├── # Arquivos de músicas (.mp3)
|       └── # Arquivos de imagem (.jpg)
├── src/                 # Código fonte
│   ├── components/      # Componentes React
│   ├── engines/         # Motores principais
│   ├── models/          # Modelos de dados
│   ├── utils/           # Utilitários
│   └── App.js           # Componente principal
└── package.json
```

## Desafios Técnicos

- **Precisão na Detecção de Tom**: Implementação de algoritmos de autocorrelação otimizados
- **Latência de Áudio**: Ajustes para reduzir e compensar a latência em navegadores
- **Sincronização Precisa**: Manter áudio, vídeo e notas perfeitamente sincronizados
- **Carregamento de Arquivos**: Parsing eficiente de arquivos grandes

## Contribuição

Para contribuir com o projeto, por favor, siga o guia de desenvolvimento no arquivo [UltraStar_Web_Final_Guide.md](UltraStar_Web_Final_Guide.md).

## Referências

1. Código fonte do UltraStar WorldParty
2. [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
3. [Pitch Detection in JavaScript](https://github.com/cwilso/PitchDetect)
4. [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
5. [React.js Documentation](https://reactjs.org/docs)

## Licença

Este projeto mantém a mesma licença do UltraStar WorldParty original. Veja o arquivo LICENSE para mais detalhes.