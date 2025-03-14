# UltraStar Web Edition - Guia Final de Desenvolvimento

## Introdução

Este guia representa a consolidação de toda a análise feita do código fonte do UltraStar WorldParty, com foco na recriação do jogo para ambiente web. O objetivo é fornecer um caminho claro e detalhado para implementação do jogo usando tecnologias web modernas, mantendo todas as funcionalidades e a experiência de jogo do UltraStar original.

## Índice

1. [Fundamentos do UltraStar](#1-fundamentos-do-ultrastar)
2. [Arquitetura do Sistema Original](#2-arquitetura-do-sistema-original)
3. [Sistema de Timing e Sincronização](#3-sistema-de-timing-e-sincronização)
4. [Gerenciamento de Letras e Visualização](#4-gerenciamento-de-letras-e-visualização)
5. [Detecção de Tom e Avaliação](#5-detecção-de-tom-e-avaliação)
6. [Arquitetura Web Proposta](#6-arquitetura-web-proposta)
7. [Implementação Técnica](#7-implementação-técnica)
8. [Formato de Arquivo e Parsing](#8-formato-de-arquivo-e-parsing)
9. [Roadmap de Desenvolvimento](#9-roadmap-de-desenvolvimento)
10. [Desafios Técnicos e Soluções](#10-desafios-técnicos-e-soluções)

## 1. Fundamentos do UltraStar

O UltraStar é um jogo de karaokê onde os jogadores cantam junto com músicas, tentando acertar os tons e tempos corretos das notas exibidas na tela. O jogo avalia a performance do jogador em tempo real, atribuindo pontuações baseadas na precisão do tom e do tempo.

### Características Principais

- **Visualização de Notas**: Exibe notas musicais em uma linha do tempo rolante
- **Detecção de Tom**: Analisa o tom da voz do jogador em tempo real
- **Sistema de Pontuação**: Pontua com base na precisão do tom e timing
- **Efeitos Visuais**: Oferece feedback visual sobre a performance do jogador
- **Múltiplos Jogadores**: Suporta jogadores múltiplos (solo, duetos, competição)
- **Biblioteca de Músicas**: Gerencia um catálogo de músicas com letras, vídeos e áudio

## 2. Arquitetura do Sistema Original

Analisando o código fonte do UltraStar WorldParty, identificamos a seguinte arquitetura:

### Componentes Principais

O jogo original é dividido nos seguintes componentes principais:

- **Motor de Renderização**: Responsável pela exibição das letras, notas e interface gráfica
- **Motor de Áudio**: Gerencia a reprodução da música e a detecção do canto do jogador
- **Motor de Vídeo**: Gerencia a reprodução do vídeo da música (quando disponível)
- **Sistema de Letras**: Controla a exibição e o progresso das letras na tela
- **Sistema de Pontuação**: Avalia o desempenho do jogador e calcula a pontuação

### Estrutura de Diretórios Relevante

- `/src/base/`: Lógica principal do jogo, incluindo notas, letras e pontuação
  - `UBeatTimer.pas`: Controle de tempo da música
  - `ULyrics.pas`: Gerenciamento de letras
  - `UMusic.pas`: Controle de reprodução musical
  - `UNote.pas`: Gerenciamento de notas musicais e tons
  - `URecord.pas`: Detecção de voz/áudio do jogador
  - `USingScores.pas`: Cálculo de pontuação
- `/src/media/`: Gerenciamento de áudio e vídeo
  - `UAudioCore_Bass.pas`: Processamento de áudio usando BASS
  - `UMediaCore_FFmpeg.pas`: Processamento de vídeo usando FFmpeg
- `/src/screens/`: Interfaces e telas do jogo

## 3. Sistema de Timing e Sincronização

### Conceito de Beats (Batidas)

O UltraStar usa um sistema baseado em "beats" (batidas) para sincronizar todos os elementos do jogo:

#### Variáveis Fundamentais

- **BPM (Beats Por Minuto)**: Define a velocidade da música
- **GAP**: Representa um atraso inicial (em milissegundos) antes do início da música
- **CurrentBeat**: Representa a batida atual da música, calculada a partir do tempo de reprodução

#### Conversão entre Beats e Tempo

No código original, encontramos as seguintes funções para conversão:

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
```

### Implementação Web do Sistema de Beats

Para recriar este sistema em JavaScript:

```javascript
class BeatSystem {
  constructor(bpm, gap) {
    this.bpm = bpm;
    this.gap = gap; // em milissegundos
    this.startTime = 0;
  }
  
  start() {
    this.startTime = performance.now();
  }
  
  getCurrentBeat(currentTime = performance.now()) {
    const elapsedTime = (currentTime - this.startTime - this.gap) / 1000;
    return this.bpm * elapsedTime / 60;
  }
  
  getTimeFromBeat(beat) {
    return (this.gap / 1000) + (beat * 60 / this.bpm);
  }
}
```

## 4. Gerenciamento de Letras e Visualização

### Estruturas de Dados

O sistema original usa várias estruturas para gerenciar letras e palavras:

#### Linha de Letra (TLyricLine)

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
end;
```

#### Palavra (TLyricWord)

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

### Gerenciamento de Linhas

O motor de letras mantém três linhas em sua fila:
- **UpperLine**: Linha superior exibida na tela (linha atual)
- **LowerLine**: Linha inferior exibida na tela (próxima linha)
- **QueueLine**: Linha em espera (será exibida quando a linha inferior terminar)

### Efeitos Visuais

O jogo suporta vários efeitos para destacar a letra atual:
- **lfxSimple**: Apenas destaca a palavra atual
- **lfxZoom**: Aumenta o tamanho da palavra atual
- **lfxSlide**: Divide a palavra em parte cantada e parte a cantar
- **lfxBall**: Adiciona uma bola que se move sobre a palavra atual
- **lfxShift**: Desloca verticalmente a palavra atual

### Implementação Web do Sistema de Letras

```javascript
class LyricsSystem {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.currentLine = null;
    this.nextLine = null;
    this.queueLine = null;
    this.effect = 'slide'; // Efeito visual para letras: simple, zoom, slide, ball, shift
  }
  
  addLine(line) {
    if (!this.currentLine) {
      this.currentLine = line;
    } else if (!this.nextLine) {
      this.nextLine = line;
    } else {
      // Rotate lines
      this.currentLine = this.nextLine;
      this.nextLine = line;
    }
  }
  
  updateMetrics(line) {
    // Calcular tamanhos e posições das palavras
    let posX = 0;
    for (const word of line.words) {
      word.x = posX;
      // Usar canvas para medir texto
      word.width = this.ctx.measureText(word.text).width;
      posX += word.width + 5; // 5px de espaçamento entre palavras
    }
    line.width = posX;
  }
  
  drawLyrics(currentBeat) {
    // Limpar canvas
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    
    // Desenhar linhas de letras
    const upperY = 100;
    const lowerY = 200;
    
    if (this.currentLine) {
      this.drawLine(this.currentLine, upperY, true, currentBeat);
    }
    
    if (this.nextLine) {
      this.drawLine(this.nextLine, lowerY, false, currentBeat);
    }
  }
  
  drawLine(line, y, isCurrentLine, currentBeat) {
    // Implementação específica do desenho de linha com efeitos visuais
    // Varia de acordo com o efeito selecionado (simple, zoom, slide, etc.)
  }
}
```

## 5. Detecção de Tom e Avaliação

### Detecção de Tom no UltraStar Original

O UltraStar usa as seguintes técnicas para detecção de tom:

#### Análise de Tom

```pascal
// Analisa o buffer de áudio
CurrentSound := AudioInputProcessor.Sound[PlayerIndex];
CurrentSound.AnalyzeBuffer;
```

#### Ajuste de Oitava

O jogo ajusta o tom cantado para a oitava correta:

```pascal
// Move o tom do jogador para a oitava apropriada
while (CurrentSound.Tone - CurrentLineFragment.Tone > 6) do
  CurrentSound.Tone := CurrentSound.Tone - 12;
while (CurrentSound.Tone - CurrentLineFragment.Tone < -6) do
  CurrentSound.Tone := CurrentSound.Tone + 12;
```

#### Tolerância de Tom

A precisão requerida varia com o nível de dificuldade:

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

### Implementação Web de Detecção de Tom

Para implementar a detecção de tom na web, usaremos a Web Audio API:

```javascript
class PitchDetector {
  constructor(audioContext) {
    this.audioContext = audioContext;
    this.analyser = audioContext.createAnalyser();
    this.analyser.fftSize = 2048;
    this.bufferLength = this.analyser.frequencyBinCount;
    this.dataArray = new Float32Array(this.bufferLength);
  }
  
  async setupMicrophone() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      this.microphone = this.audioContext.createMediaStreamSource(stream);
      this.microphone.connect(this.analyser);
      return true;
    } catch (error) {
      console.error("Erro ao acessar microfone:", error);
      return false;
    }
  }
  
  detectPitch() {
    this.analyser.getFloatTimeDomainData(this.dataArray);
    
    // Implementar algoritmo de autocorrelação para detectar o tom
    // Podemos usar bibliotecas como Pitchy.js ou implementar autocorrelação
    
    // Algoritmo básico de autocorrelação:
    const result = this.autoCorrelate(this.dataArray, this.audioContext.sampleRate);
    
    // Converter frequência para nota MIDI
    if (result !== -1) {
      return this.freqToMidi(result);
    }
    return -1;
  }
  
  autoCorrelate(buffer, sampleRate) {
    // Implementação do algoritmo de autocorrelação
    // Retorna a frequência fundamental ou -1 se não for detectada
  }
  
  freqToMidi(frequency) {
    // Converte frequência (Hz) para nota MIDI
    // A4 (Lá 440Hz) = MIDI 69
    return Math.round(12 * (Math.log(frequency / 440) / Math.log(2))) + 69;
  }
  
  adjustOctave(detectedNote, expectedNote) {
    // Ajusta a oitava assim como no UltraStar original
    while (detectedNote - expectedNote > 6) {
      detectedNote -= 12;
    }
    while (detectedNote - expectedNote < -6) {
      detectedNote += 12;
    }
    return detectedNote;
  }
}
```

### Sistema de Pontuação

O sistema de pontuação no UltraStar depende do tipo da nota e da precisão:

```javascript
class ScoringSystem {
  constructor() {
    this.maxSongPoints = 10000;
    this.maxLineBonus = 1000;
    this.scoreFactors = {
      normal: 1,
      golden: 2,
      rap: 1,
      rapGolden: 2
    };
    
    this.playerScores = {
      normalScore: 0,
      goldenScore: 0,
      lineBonus: 0
    };
  }
  
  calculateNoteScore(noteType, songTotalValue) {
    return (this.maxSongPoints / songTotalValue) * this.scoreFactors[noteType];
  }
  
  updateScore(noteType, points) {
    if (noteType === 'normal' || noteType === 'rap') {
      this.playerScores.normalScore += points;
    } else if (noteType === 'golden' || noteType === 'rapGolden') {
      this.playerScores.goldenScore += points;
    }
    
    // Arredondar pontuações para exibição
    const normalScoreInt = Math.round(this.playerScores.normalScore / 10) * 10;
    const goldenScoreInt = Math.round(this.playerScores.goldenScore / 10) * 10;
    
    return {
      normalScore: normalScoreInt,
      goldenScore: goldenScoreInt,
      totalScore: normalScoreInt + goldenScoreInt + this.playerScores.lineBonus
    };
  }
}
```

## 6. Arquitetura Web Proposta

### Tecnologias Recomendadas

1. **Frontend**:
   - **Framework**: React.js para gerenciamento de componentes
   - **Renderização**: HTML5 Canvas para desenho das letras e notas
   - **Áudio**: Web Audio API
   - **Detecção de Tom**: Algoritmo de autocorrelação personalizado ou biblioteca Pitchy.js

2. **Backend** (opcional para versão multiplayer):
   - **API**: Node.js + Express
   - **Banco de Dados**: MongoDB para armazenar músicas, letras e pontuações
   - **Armazenamento**: Possível integração com armazenamento em nuvem para arquivos de mídia

### Arquitetura de Componentes

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

## 7. Implementação Técnica

### Inicialização do Projeto

#### Estrutura de Arquivos

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

#### Configuração Inicial

```bash
# Criar aplicação React
npx create-react-app ultrastar-web

# Instalar dependências úteis
npm install howler pitchy webfontloader

# Iniciar servidor de desenvolvimento
npm start
```

### Implementação do Core do Jogo

O núcleo do sistema deve incluir as seguintes classes:

#### 1. GameEngine

```javascript
class GameEngine {
  constructor() {
    this.isPlaying = false;
    this.audioEngine = new AudioEngine();
    this.beatSystem = null;
    this.lyricsRenderer = null;
    this.pitchDetector = null;
    this.scoringSystem = new ScoringSystem();
    this.currentSong = null;
    this.currentBeat = 0;
    this.animationFrameId = null;
  }
  
  async loadSong(songId) {
    // Carrega informações da música, letra e áudio
    const songData = await SongLoader.load(songId);
    this.currentSong = songData;
    
    // Inicializar subsistemas
    this.beatSystem = new BeatSystem(songData.bpm, songData.gap);
    await this.audioEngine.loadSong(songData.audioUrl);
    this.lyricsRenderer.loadLyrics(songData.lyrics);
    
    return true;
  }
  
  async startGame() {
    // Iniciar microfone
    await this.pitchDetector.setupMicrophone();
    
    // Iniciar música
    this.audioEngine.play();
    this.beatSystem.start();
    this.isPlaying = true;
    
    // Iniciar loop principal
    this.gameLoop();
  }
  
  gameLoop() {
    if (!this.isPlaying) return;
    
    // Obter beat atual
    this.currentBeat = this.beatSystem.getCurrentBeat();
    
    // Atualizar letras
    this.lyricsRenderer.update(this.currentBeat);
    
    // Detectar tom do jogador
    if (this.pitchDetector) {
      const detectedNote = this.pitchDetector.detectPitch();
      if (detectedNote > 0) {
        // Processar nota detectada
        this.processPlayerNote(detectedNote, this.currentBeat);
      }
    }
    
    // Renderizar tudo
    this.render();
    
    // Próximo frame
    this.animationFrameId = requestAnimationFrame(this.gameLoop.bind(this));
  }
  
  processPlayerNote(detectedNote, currentBeat) {
    // Obter nota esperada neste momento
    const expectedNote = this.currentSong.getNoteAtBeat(currentBeat);
    
    if (expectedNote) {
      // Ajustar oitava
      const adjustedNote = this.pitchDetector.adjustOctave(detectedNote, expectedNote.tone);
      
      // Verificar tolerância baseada na dificuldade
      const tolerance = this.getDifficultyTolerance();
      const isHit = Math.abs(adjustedNote - expectedNote.tone) <= tolerance;
      
      if (isHit) {
        // Calcular pontuação
        const points = this.scoringSystem.calculateNoteScore(
          expectedNote.type,
          this.currentSong.totalValue
        );
        
        // Atualizar pontuação
        const newScore = this.scoringSystem.updateScore(expectedNote.type, points);
        
        // Notificar acerto da nota
        this.onNoteHit(expectedNote, newScore);
      }
    }
  }
  
  getDifficultyTolerance() {
    // Dificuldade: 0 (fácil) a 2 (difícil)
    const difficulty = 1; // Médio por padrão
    return 2 - difficulty;
  }
  
  render() {
    // Limpar canvas
    this.lyricsRenderer.clear();
    
    // Renderizar letras e notas
    this.lyricsRenderer.draw(this.currentBeat);
  }
  
  endGame() {
    // Parar loop e liberar recursos
    this.isPlaying = false;
    cancelAnimationFrame(this.animationFrameId);
    this.audioEngine.stop();
    
    // Mostrar pontuação final
    return this.scoringSystem.getFinalScore();
  }
}
```

## 8. Formato de Arquivo e Parsing

### Formato do Arquivo UltraStar (.txt)

O UltraStar usa arquivos de texto (.txt) para definir as letras e notas:

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

Cada linha começa com um prefixo que indica seu tipo:
- `:` - Nota normal
- `*` - Nota freestyle (não avaliada por tom)
- `F` - Nota dourada (pontuação dupla)
- `-` - Fim de linha/pausa

Após o prefixo, temos:
1. Beat de início
2. Duração em beats
3. Tom da nota (valor MIDI)
4. Texto da sílaba/palavra

### Parser de Arquivos UltraStar

```javascript
class UltraStarParser {
  static parse(fileContent) {
    const lines = fileContent.trim().split('\n');
    const metadata = {};
    const notes = [];
    
    lines.forEach(line => {
      line = line.trim();
      
      // Metadata
      if (line.startsWith('#')) {
        const [key, value] = line.substring(1).split(':');
        metadata[key.toLowerCase()] = value;
      } 
      // Notes
      else {
        const match = line.match(/^([:\*F\-]) (\d+) (\d+) (\d+) (.*)$/);
        if (match) {
          const [_, type, startBeat, length, tone, text] = match;
          
          notes.push({
            type: this.getNoteType(type),
            startBeat: parseInt(startBeat),
            length: parseInt(length),
            tone: parseInt(tone),
            text
          });
        }
      }
    });
    
    // Organizar notas em linhas
    const songLines = this.organizeIntoLines(notes);
    
    return {
      metadata,
      bpm: parseFloat(metadata.bpm || "120"),
      gap: parseInt(metadata.gap || "0"),
      videoGap: parseInt(metadata.videogap || "0"),
      lines: songLines,
      totalValue: this.calculateTotalValue(notes)
    };
  }
  
  static getNoteType(prefix) {
    switch (prefix) {
      case ':': return 'normal';
      case '*': return 'freestyle';
      case 'F': return 'golden';
      case '-': return 'pause';
      default: return 'normal';
    }
  }
  
  static organizeIntoLines(notes) {
    // Organiza notas em linhas baseadas em pausas ou fim de linha
    // ...código de organização
  }
  
  static calculateTotalValue(notes) {
    // Calcula valor total da música para pontuação
    // ...código de cálculo de valor total
  }
}
```

## 9. Roadmap de Desenvolvimento

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

## 10. Desafios Técnicos e Soluções

### Desafio 1: Precisão na Detecção de Tom

**Problema**: A Web Audio API tem limitações para detecção precisa de frequência em tempo real.

**Soluções**:
- Implementar um algoritmo de autocorrelação otimizado para JavaScript
- Usar fftSize maior no AnalyserNode para melhor resolução de frequência
- Aplicar filtros para reduzir ruído e interferências
- Implementar sistema de calibração de microfone

### Desafio 2: Latência de Áudio

**Problema**: Navegadores têm latência variável no processamento de áudio de microfone.

**Soluções**:
- Usar bufferSize menor para reduzir latência no ScriptProcessorNode
- Implementar sistema de calibração para medir e compensar a latência
- Ajustar o algoritmo de sincronização para levar em conta a latência
- Utilizar Web Audio Worklet quando disponível para processamento mais eficiente

### Desafio 3: Sincronização Precisa

**Problema**: Manter a sincronização precisa entre áudio, vídeo e notas.

**Soluções**:
- Basear toda a sincronização no currentTime da Web Audio API
- Implementar um sistema de correção contínua de drift
- Usar requestAnimationFrame com timestamp para sincronização visual
- Manter um sistema de clock central para o jogo inteiro

### Desafio 4: Carregamento e Parsing de Arquivos

**Problema**: Carregar e parsear arquivos grandes de forma eficiente.

**Soluções**:
- Implementar carregamento streaming para áudio e vídeo
- Usar Web Workers para parsing de arquivos sem bloquear a UI
- Implementar cache de arquivos usando IndexedDB
- Criar formato otimizado (JSON) para letras e notas em vez de parsear TXT em tempo real

---

## Conclusão

Este guia fornece todas as informações necessárias para reconstruir o UltraStar como uma aplicação web moderna. Seguindo a arquitetura e implementações propostas, você poderá criar uma versão web fiel ao jogo original, mantendo todas as suas funcionalidades principais.

O UltraStar Web Edition pode não apenas replicar a experiência do jogo original, mas também expandir suas capacidades aproveitando as tecnologias web modernas, como compartilhamento online, integração com serviços de streaming e adaptação para dispositivos móveis.

---

## Referências

1. Código fonte do UltraStar WorldParty
2. Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
3. Pitch Detection in JavaScript: https://github.com/cwilso/PitchDetect
4. Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
5. React.js Documentation: https://reactjs.org/docs