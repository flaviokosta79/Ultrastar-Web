# Guia de Desenvolvimento - UltraStar Web Edition

## Visão Geral do Projeto

Este documento contém informações detalhadas sobre como o UltraStar funciona e como recriá-lo para a web. O objetivo é fornecer um guia completo para desenvolvedores que queiram criar uma versão web do popular jogo de karaokê UltraStar WorldParty, mantendo todas as funcionalidades principais do jogo original.

## Índice

1. [Arquitetura Geral do UltraStar](#1-arquitetura-geral-do-ultrastar)
2. [Sistema de Timing e Sincronização](#2-sistema-de-timing-e-sincronização)
3. [Sistema de Letras e Visualização](#3-sistema-de-letras-e-visualização)
4. [Detecção de Tom e Avaliação](#4-detecção-de-tom-e-avaliação)
5. [Gerenciamento de Áudio e Vídeo](#5-gerenciamento-de-áudio-e-vídeo)
6. [Arquitetura Proposta para Versão Web](#6-arquitetura-proposta-para-versão-web)
7. [Estrutura de Arquivos e Formatos](#7-estrutura-de-arquivos-e-formatos)
8. [Roadmap de Desenvolvimento](#8-roadmap-de-desenvolvimento)
9. [Desafios e Considerações](#9-desafios-e-considerações)

## 1. Arquitetura Geral do UltraStar

O UltraStar é um jogo de karaokê onde os jogadores cantam junto com uma música, tentando acertar os tons e tempos corretos das notas exibidas na tela. O jogo foi originalmente desenvolvido em Delphi/Pascal e utiliza diversas bibliotecas para gerenciamento de mídia, gráficos e detecção de áudio.

### Componentes Principais

O jogo é dividido em vários componentes principais:

- **Motor de Renderização**: Responsável pela exibição das letras, notas e interface gráfica
- **Motor de Áudio**: Gerencia a reprodução da música e a detecção do canto do jogador
- **Motor de Vídeo**: Gerencia a reprodução do vídeo da música (quando disponível)
- **Sistema de Letras**: Controla a exibição e o progresso das letras na tela
- **Sistema de Pontuação**: Avalia o desempenho do jogador e calcula a pontuação

### Estrutura de Diretórios Relevante

Os diretórios mais importantes para entender o funcionamento do jogo são:

- `/src/base/`: Contém a lógica principal do jogo, incluindo gerenciamento de notas, letras e pontuação
- `/src/media/`: Gerencia a reprodução de áudio e vídeo
- `/src/screens/`: Contém as diferentes telas e interfaces do jogo

## 2. Sistema de Timing e Sincronização

### Conceito de Beats (Batidas)

O UltraStar usa um sistema baseado em "beats" (batidas) para sincronizar todos os elementos do jogo. Um beat é a unidade fundamental de tempo musical no jogo.

#### Variáveis e Funções Importantes:

- **BPM (Beats Por Minuto)**: Define a velocidade da música
- **GAP**: Representa um atraso inicial (em milissegundos) antes do início da música
- **CurrentBeat**: Representa a batida atual da música, calculada a partir do tempo de reprodução

#### Conversão entre Beats e Tempo:

```pascal
// Converte beats em tempo (segundos)
function GetTimeForBeats(BPM, Beats: real): real;
begin
  Result := 60 / BPM * Beats;
end;

// Converte tempo em beats
function GetBeats(BPM, msTime: real): real;
begin
  Result := BPM * msTime / 60;
end;

// Obtém o tempo a partir de um beat específico
function GetTimeFromBeat(Beat: integer; SelfSong: TSong = nil): real;
var
  Song: TSong;
begin
  if (SelfSong <> nil) then
    Song := SelfSong
  else
    Song := CurrentSong;
  Result := Song.GAP / 1000 + Beat * 60 / Song.BPM;
end;
```

### Sincronização

A sincronização é mantida através da atualização constante do estado das letras usando `LyricsState.UpdateBeats()`, que calcula o beat atual com base no tempo de reprodução da música.

O jogo possui diferentes "beats" para diferentes propósitos:
- **CurrentBeat**: Beat principal para a sincronização geral
- **CurrentBeatC**: Beat para cliques e assistência de ritmo
- **CurrentBeatD**: Beat para detecção de notas

## 3. Sistema de Letras e Visualização

### Estruturas de Dados

O sistema de letras é composto por várias estruturas:

#### TLyricLine (Linha de Letra)
```pascal
TLyricLine = class
  public
    Text:           UTF8String;   // texto da linha
    Width:          real;         // largura
    Height:         real;         // altura
    Words:          TLyricWordArray;   // palavras nesta linha
    CurWord:        integer;      // índice da palavra atual ativa
    Start:          integer;      // início desta linha em beats
    StartNote:      integer;      // início da primeira nota desta linha em beats
    Length:         integer;      // duração em beats
    Players:        byte;         // jogadores que devem cantar esta linha
    LastLine:       boolean;      // é a última linha da música?
    // Métodos...
end;
```

#### TLyricWord (Palavra)
```pascal
TLyricWord = record
  X:          real;     // posição X (canto esquerdo)
  Width:      real;     // largura
  Start:      Longint;  // início da palavra em beats
  Length:     Longint;  // duração da palavra em beats
  Text:       UTF8String; // texto
  Freestyle:  boolean;  // é freestyle?
end;
```

#### TNote (Nota)
```pascal
TPlayerNote = record
  Start:    integer;   // início em beats
  Length:   integer;   // duração em beats
  Detect:   real;      // posição precisa detectada na nota
  Tone:     real;      // tom da nota
  Perfect:  boolean;   // verdadeiro se a nota corresponde perfeitamente à original
  Hit:      boolean;   // verdadeiro se a nota acertou a linha
  NoteType: TNoteType; // tipo da nota (normal, dourada, etc)
end;
```

### Motor de Renderização de Letras

O UltraStar usa OpenGL para renderizar as letras. A renderização é feita em etapas:

1. **Cálculo de Métricas**: `UpdateLineMetrics` calcula o tamanho e posicionamento das palavras
2. **Renderização de Linhas**: `DrawLyricsLine` desenha as linhas, destacando a palavra atual
3. **Efeitos Visuais**: O jogo suporta vários efeitos para destacar a letra atual:
   - **lfxSimple**: Apenas destaca a palavra atual
   - **lfxZoom**: Aumenta o tamanho da palavra atual
   - **lfxSlide**: Divide a palavra em parte cantada e parte a cantar
   - **lfxBall**: Adiciona uma bola que se move sobre a palavra atual
   - **lfxShift**: Desloca verticalmente a palavra atual

### Gerenciamento de Linhas

O motor de letras mantém três linhas em sua fila:
- **UpperLine**: Linha superior exibida na tela (linha atual)
- **LowerLine**: Linha inferior exibida na tela (próxima linha)
- **QueueLine**: Linha em espera (será exibida quando a linha inferior terminar)

## 4. Detecção de Tom e Avaliação

### Captura e Análise de Áudio

O jogo captura o áudio do microfone e o analisa para determinar o tom cantado pelo jogador:

```pascal
// Analisa o buffer de áudio
CurrentSound := AudioInputProcessor.Sound[PlayerIndex];
CurrentSound.AnalyzeBuffer;
```

### Comparação de Tom

Quando uma nota é detectada, o jogo compara o tom cantado com o tom esperado. Para fazer isso, ele primeiro ajusta o tom para a oitava correta:

```pascal
// Move o tom do jogador para a oitava apropriada
while (CurrentSound.Tone - CurrentLineFragment.Tone > 6) do
  CurrentSound.Tone := CurrentSound.Tone - 12;
while (CurrentSound.Tone - CurrentLineFragment.Tone < -6) do
  CurrentSound.Tone := CurrentSound.Tone + 12;
```

### Tolerância e Dificuldade

A precisão requerida para acertar uma nota varia conforme o nível de dificuldade:

```pascal
if (ScreenSong.Mode = smNormal) then
  Range := 2 - Ini.PlayerLevel[PlayerIndex]
else
  Range := 2 - Ini.Difficulty;

// Verifica se o jogador acertou o tom correto dentro da faixa tolerada
if (Abs(CurrentLineFragment.Tone - CurrentSound.Tone) <= Range) or 
   (CurrentLineFragment.NoteType = ntRap) or 
   (CurrentLineFragment.NoteType = ntRapGolden) then
  // Nota acertada
```

### Sistema de Pontuação

O sistema de pontuação depende do tipo de nota e da precisão:

```pascal
// Tipos de notas e seus fatores de pontuação
ScoreFactor[ntNormal] := 1;
ScoreFactor[ntGolden] := 2;
ScoreFactor[ntRap] := 1;
ScoreFactor[ntRapGolden] := 2;

// Cálculo da pontuação para uma nota
CurNotePoints := (MaxSongPoints / CurrentSong.Lines[CP].ScoreValue) * ScoreFactor[CurrentLineFragment.NoteType];

// Atualização da pontuação do jogador
case CurrentLineFragment.NoteType of
  ntNormal:    CurrentPlayer.Score       := CurrentPlayer.Score       + CurNotePoints;
  ntGolden:    CurrentPlayer.ScoreGolden := CurrentPlayer.ScoreGolden + CurNotePoints;
  ntRap:       CurrentPlayer.Score       := CurrentPlayer.Score       + CurNotePoints;
  ntRapGolden: CurrentPlayer.ScoreGolden := CurrentPlayer.ScoreGolden + CurNotePoints;
end;
```

## 5. Gerenciamento de Áudio e Vídeo

### Reprodução de Áudio

O UltraStar usa a biblioteca BASS para processamento e reprodução de áudio. Isso é visível nos arquivos como `UAudioCore_Bass.pas` e `UAudioPlayback_Bass.pas`.

### Reprodução de Vídeo

O jogo utiliza FFmpeg para a reprodução de vídeo, conforme indicam arquivos como `UMediaCore_FFmpeg.pas`.

### Sincronização de Áudio/Vídeo com Letras

A sincronização acontece através do sistema de beats, onde tanto o áudio quanto as letras estão sincronizados com base no tempo calculado a partir dos beats.

## 6. Arquitetura Proposta para Versão Web

### Tecnologias Recomendadas

1. **Frontend**:
   - **Framework**: React ou Vue.js para gerenciamento de componentes
   - **Renderização**: HTML5 Canvas ou WebGL (via Three.js ou Pixi.js)
   - **Áudio**: Web Audio API
   - **Detecção de Tom**: PitchDetect.js ou bibliotecas similares

2. **Backend**:
   - **API**: Node.js + Express ou qualquer framework REST
   - **Banco de Dados**: MongoDB ou PostgreSQL para armazenar músicas, letras e pontuações
   - **Armazenamento**: Solução cloud (AWS S3, Google Cloud Storage) para arquivos de mídia

### Componentes Principais

#### 1. Motor de Web Audio
Responsável por:
- Carregar e reproduzir música
- Capturar áudio do microfone
- Analisar o tom cantado em tempo real

```javascript
class AudioEngine {
  constructor() {
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    this.analyser = this.audioContext.createAnalyser();
    // Configuração adicional...
  }

  startMicrophoneCapture() {
    // Solicitar acesso ao microfone
    navigator.mediaDevices.getUserMedia({ audio: true })
      .then(stream => {
        this.microphoneSource = this.audioContext.createMediaStreamSource(stream);
        this.microphoneSource.connect(this.analyser);
        // Iniciar detecção de tom
      })
      .catch(err => console.error("Erro ao acessar microfone:", err));
  }

  detectPitch() {
    // Implementação de algoritmo de detecção de tom
    // Pode usar bibliotecas como ML5.js, PitchDetect.js ou algoritmos personalizados
  }
}
```

#### 2. Motor de Renderização
Responsável por:
- Exibir letras e notas na tela
- Aplicar efeitos visuais
- Renderizar interface do usuário

```javascript
class LyricsRenderer {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.currentLine = null;
    this.nextLine = null;
    this.queueLine = null;
  }

  updateMetrics(line) {
    // Calcular tamanhos e posições das palavras
  }

  drawLyrics(currentBeat) {
    // Desenhar linhas de letras
    this.drawLine(this.currentLine, true, currentBeat);
    this.drawLine(this.nextLine, false, currentBeat);
  }

  drawLine(line, isCurrentLine, currentBeat) {
    // Implementar desenho de linha com efeitos visuais
  }
}
```

#### 3. Sistema de Timing e Sincronização
```javascript
class BeatSystem {
  constructor(bpm, gap) {
    this.bpm = bpm;
    this.gap = gap;
    this.startTime = 0;
  }

  getCurrentBeat(currentTime) {
    const elapsedTime = currentTime - this.startTime - (this.gap / 1000);
    return this.bpm * elapsedTime / 60;
  }

  getTimeFromBeat(beat) {
    return this.gap / 1000 + beat * 60 / this.bpm;
  }
}
```

#### 4. Gerenciador de Notas e Avaliação
```javascript
class NoteManager {
  constructor(beatSystem) {
    this.beatSystem = beatSystem;
    this.currentLine = 0;
    this.currentWord = 0;
    this.playerNotes = [];
  }

  addPlayerNote(beat, tone, isHit) {
    // Adicionar uma nova nota do jogador
  }

  evaluatePerformance(detectedTone, currentBeat, difficultyLevel) {
    // Verificar se o jogador está acertando a nota atual
    // Calcular pontuação com base na precisão
  }
}
```

## 7. Estrutura de Arquivos e Formatos

### Formato dos Arquivos de Música

O UltraStar usa arquivos de texto (.txt) para definir as letras, notas e timing de cada música. A seguir está a estrutura básica desses arquivos:

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
...
```

Cada linha começa com `:` seguido por:
1. O tempo de início da nota (em beats)
2. A duração da nota (em beats)
3. O tom da nota (em semitons - 0-118, onde 60 é o Dó central)
4. A sílaba ou palavra correspondente

Notas especiais:
- Linhas que começam com `*` são notas freestyle (não são avaliadas quanto ao tom)
- Linhas que começam com `F` são notas douradas (valem o dobro de pontos)
- A letra `E` após os números indica fim de linha
- A letra `P` indica pausa

### Estrutura Proposta para Versão Web

#### Diretório do Projeto
```
ultrastar-web/
├── public/              # Arquivos estáticos
│   ├── songs/           # Arquivos de músicas (.mp3)
│   ├── videos/          # Arquivos de vídeos (.mp4)
│   └── lyrics/          # Arquivos de letras (.txt ou .json)
├── src/                 # Código fonte
│   ├── components/      # Componentes React/Vue
│   │   ├── SongSelect/
│   │   ├── SingScreen/
│   │   └── ScoreScreen/
│   ├── engines/         # Motores principais
│   │   ├── audioEngine.js
│   │   ├── lyricsEngine.js
│   │   ├── beatSystem.js
│   │   └── scoreEngine.js
│   ├── models/          # Modelos de dados
│   │   ├── Song.js
│   │   ├── Line.js
│   │   └── Note.js
│   ├── utils/           # Utilitários
│   │   ├── pitchDetection.js
│   │   ├── fileParser.js
│   │   └── audioAnalysis.js
│   └── App.js           # Componente principal
├── server/              # Código do servidor (opcional)
│   ├── api/
│   └── db/
└── package.json
```

## 8. Roadmap de Desenvolvimento

### Fase 1: Infraestrutura Básica
1. Configurar o projeto com React/Vue e ferramentas necessárias
2. Implementar leitor de arquivos de letra (.txt) do UltraStar
3. Implementar sistema básico de beats e timing
4. Criar componente de reprodução de áudio

### Fase 2: Motor de Visualização
1. Desenvolver renderização básica de letras
2. Implementar desenho de linhas de notas
3. Adicionar efeitos visuais para palavras (similar aos efeitos do UltraStar)
4. Integrar reprodução de vídeo

### Fase 3: Motor de Áudio e Detecção
1. Implementar captura de microfone
2. Desenvolver algoritmo de detecção de tom
3. Criar sistema de comparação entre tom cantado e tom esperado
4. Implementar sistema de pontuação

### Fase 4: Integração e Polimento
1. Integrar todos os componentes
2. Implementar seleção de músicas e interface de usuário
3. Adicionar suporte para múltiplos jogadores
4. Polir interface e experiência do usuário

### Fase 5: Expansão
1. Implementar sistema de playlists
2. Adicionar modos de jogo adicionais (duetos, etc.)
3. Implementar editor de letras/notas online
4. Adicionar recursos sociais (compartilhamento de pontuação, etc.)

## 9. Desafios e Considerações

### Desafios Técnicos

1. **Precisão da Detecção de Tom**: A Web Audio API tem limitações em comparação com bibliotecas nativas. Será necessário otimizar os algoritmos de detecção de tom para garantir precisão adequada.

2. **Latência**: A latência do áudio no navegador pode ser maior do que em aplicativos desktop. Será necessário implementar compensações para garantir a sincronização correta.

3. **Compatibilidade de Navegadores**: Diferentes navegadores podem ter implementações variadas das APIs de áudio e vídeo. Será necessário testar extensivamente e fornecer fallbacks quando necessário.

4. **Desempenho**: A renderização de efeitos visuais e a análise de áudio em tempo real podem ser intensivas para o CPU. Otimizações serão necessárias para garantir um bom desempenho.

### Considerações de Design

1. **Responsividade**: O design deve ser adaptável para diferentes tamanhos de tela, incluindo dispositivos móveis.

2. **Acessibilidade**: Incluir opções de acessibilidade, como ajustes de contraste e tamanho de texto.

3. **Experiência do Usuário**: Manter a experiência divertida e envolvente do UltraStar original, enquanto adapta-se ao ambiente da web.

### Limitações

1. **Acesso ao Microfone**: Os navegadores requerem permissão explícita para acessar o microfone, o que pode criar fricção na experiência do usuário.

2. **Tamanho dos Arquivos**: Arquivos de música e vídeo podem ser grandes, o que pode causar problemas de carregamento. Considere implementar streaming progressivo.

3. **Compatibilidade com Formatos**: Nem todos os navegadores suportam os mesmos formatos de áudio e vídeo. Forneça múltiplos formatos ou conversão automatizada.

---

Este documento serve como um guia abrangente para o desenvolvimento de uma versão web do UltraStar. Ele fornece informações detalhadas sobre como o jogo original funciona e como essas funcionalidades podem ser recriadas usando tecnologias web modernas. Use este guia como ponto de partida para o desenvolvimento do seu projeto UltraStar Web Edition.