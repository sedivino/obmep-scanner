# 📱 OBMEP Scanner

> Aplicativo mobile para correção automática dos cartões-resposta da OBMEP via visão computacional

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Android%20%7C%20iOS-brightgreen.svg)]()
[![Status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 📋 Índice

- [Sobre o projeto](#-sobre-o-projeto)
- [Funcionalidades](#-funcionalidades)
- [Fluxo do aplicativo](#-fluxo-do-aplicativo)
- [Como funciona a visão computacional](#-como-funciona-a-visão-computacional)
- [Tecnologias](#-tecnologias)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Como usar](#-como-usar)
- [Roadmap](#-roadmap)
- [Contribuindo](#-contribuindo)
- [Licença](#-licença)

---

## 📖 Sobre o projeto

A **OBMEP** (Olimpíada Brasileira de Matemática das Escolas Públicas), realizada pelo IMPA, aplica anualmente provas de múltipla escolha para alunos do ensino fundamental e médio de escolas públicas em todo o Brasil. Os cartões-resposta utilizados na competição possuem estrutura padronizada com marcadores fiduciais (bolinhas pretas nos cantos) e grade de bolhas de A a E para 20 questões, distribuídas nos Níveis 1, 2 e 3.

O **OBMEP Scanner** é um aplicativo mobile que permite ao professor:

1. Cadastrar o gabarito oficial da prova uma única vez
2. Fotografar os cartões-resposta dos alunos com a câmera do celular
3. Receber automaticamente a quantidade de acertos de cada aluno

O processo elimina a correção manual, reduzindo erros e economizando tempo — especialmente útil em escolas com grande número de participantes.

---

## ✨ Funcionalidades

- **Entrada de gabarito única** — o professor digita as respostas corretas (A–E) para as 20 questões apenas uma vez por sessão
- **Leitura automática do cartão** — a câmera detecta e processa o cartão-resposta sem necessidade de posicionamento preciso
- **Correção instantânea** — o app exibe quantas questões o aluno acertou imediatamente após a leitura
- **Sessão contínua** — o professor pode corrigir cartões sequencialmente até encerrar a sessão manualmente
- **Suporte a múltiplos níveis** — compatível com os cartões dos Níveis 1, 2 e 3 da OBMEP
- **Feedback visual** — destaque das questões corretas e incorretas na tela de resultado
- **Sem necessidade de internet** — todo o processamento é feito localmente no dispositivo

---

## 🔄 Fluxo do aplicativo

```
┌─────────────────────────────────────────────────────────┐
│                    INÍCIO DA SESSÃO                     │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│               TELA DE ENTRADA DO GABARITO               │
│  Professor digita as respostas corretas (Q1 a Q20)      │
│  Exemplo: A B C D E A B C D E A B C D E A B C D E      │
└────────────────────────┬────────────────────────────────┘
                         │  (feito apenas uma vez)
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  TELA DE ESCANEAMENTO                   │
│  Professor aponta a câmera para o cartão do aluno       │
│  O app detecta automaticamente o cartão                 │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  TELA DE RESULTADO                      │
│  Exibe: nome do aluno (via OCR, se disponível)          │
│  Exibe: X / 20 acertos                                  │
│  Destaca questões corretas ✅ e incorretas ❌           │
└────────────────────────┬────────────────────────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
     [Próximo aluno]           [Encerrar]
    (volta ao scanner)     (encerra a sessão)
```

---

## 🔬 Como funciona a visão computacional

O cartão-resposta da OBMEP possui características que facilitam a detecção automática:

### 1. Detecção de marcadores fiduciais
Os quatro círculos pretos nos cantos do cartão servem como pontos de referência. A biblioteca **OpenCV** os detecta usando:
- Limiarização adaptativa (binarização)
- Detecção de contornos circulares (`HoughCircles` ou `findContours`)

### 2. Correção de perspectiva (homografia)
A partir dos 4 marcadores, o app calcula uma transformação de perspectiva (`getPerspectiveTransform`) que "achata" o cartão mesmo que a foto tenha sido tirada em ângulo.

### 3. Leitura das bolhas
Com o cartão normalizado, o app divide a grade em 20 × 5 células (20 questões × 5 alternativas). Para cada célula, calcula a proporção de pixels escuros: a célula com maior preenchimento é a alternativa marcada pelo aluno.

### 4. Comparação com o gabarito
As respostas detectadas são comparadas com o gabarito armazenado. O resultado é computado e exibido imediatamente.

```
Imagem da câmera
      │
      ▼
Binarização (escala de cinza + threshold)
      │
      ▼
Detecção dos 4 marcadores nos cantos
      │
      ▼
Transformação de perspectiva (warp)
      │
      ▼
Segmentação da grade 20×5
      │
      ▼
Leitura de cada bolha (% pixels preenchidos)
      │
      ▼
Comparação com o gabarito
      │
      ▼
Resultado: X / 20 acertos
```

---

## 🛠 Tecnologias

| Camada | Tecnologia | Motivo da escolha |
|---|---|---|
| Framework mobile | [React Native](https://reactnative.dev/) | Suporte a Android e iOS com uma única base de código |
| Visão computacional | [OpenCV](https://opencv.org/) via módulo nativo | Biblioteca mais robusta para detecção de formas e homografia |
| Câmera | [react-native-vision-camera](https://mrousavy.com/react-native-vision-camera/) | Acesso de baixa latência à câmera com suporte a frame processors |
| Navegação | [React Navigation](https://reactnavigation.org/) | Padrão da comunidade React Native |
| Gerenciamento de estado | [Zustand](https://zustand-demo.pmnd.rs/) | Leve e simples para o escopo do projeto |
| Linguagem | TypeScript | Tipagem estática reduz erros em tempo de desenvolvimento |

---

## 📁 Estrutura do projeto

```
obmep-scanner/
├── src/
│   ├── screens/
│   │   ├── GabaritoScreen.tsx      # Tela de entrada do gabarito
│   │   ├── ScannerScreen.tsx       # Tela de leitura do cartão
│   │   └── ResultadoScreen.tsx     # Tela de exibição do resultado
│   ├── services/
│   │   ├── cardDetector.ts         # Detecção de marcadores e warp
│   │   ├── bubbleReader.ts         # Leitura das bolhas marcadas
│   │   └── grader.ts               # Comparação com o gabarito
│   └── utils/
│       ├── imageUtils.ts           # Funções auxiliares de imagem
│       └── constants.ts            # Constantes (grid, thresholds)
├── assets/
│   └── samples/                    # Imagens de cartões para testes
├── docs/
│   ├── architecture.md             # Diagrama de arquitetura detalhado
│   └── vision-pipeline.md          # Documentação do pipeline de visão
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── workflows/
│       └── ci.yml                  # Pipeline de CI (lint + testes)
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── LICENSE
└── README.md
```

---

## ✅ Pré-requisitos

- [Node.js](https://nodejs.org/) 18 ou superior
- [React Native CLI](https://reactnative.dev/docs/environment-setup)
- Android Studio (para Android) ou Xcode (para iOS)
- Java Development Kit (JDK) 17
- Dispositivo físico recomendado para testes de câmera

---

## 🚀 Instalação

```bash
# 1. Clone o repositório
git clone https://github.com/SEU_USERNAME/obmep-scanner.git

# 2. Entre na pasta
cd obmep-scanner

# 3. Instale as dependências
npm install

# 4. Instale as dependências nativas (iOS)
cd ios && pod install && cd ..

# 5. Inicie o app no Android
npx react-native run-android

# 6. Ou inicie no iOS
npx react-native run-ios
```

---

## 📱 Como usar

**1. Inserir o gabarito**
- Abra o app e toque em "Novo gabarito"
- Selecione o nível da prova (1, 2 ou 3)
- Para cada questão (1 a 20), selecione a alternativa correta (A, B, C, D ou E)
- Toque em "Confirmar gabarito"

**2. Corrigir um cartão**
- Posicione o cartão-resposta do aluno sobre uma superfície plana com boa iluminação
- Aponte a câmera para o cartão — o app detecta automaticamente os marcadores
- Aguarde o processamento (geralmente menos de 1 segundo)

**3. Ver o resultado**
- O app exibe o número de acertos e as questões corretas/incorretas
- Toque em "Próximo aluno" para corrigir o cartão seguinte
- Toque em "Encerrar sessão" para finalizar

---

## 🗺 Roadmap

- [x] Definição da arquitetura do projeto
- [x] Estrutura inicial do repositório
- [ ] **v0.1 — MVP**
  - [ ] Tela de entrada do gabarito
  - [ ] Captura de imagem via câmera
  - [ ] Detecção dos marcadores fiduciais
  - [ ] Correção de perspectiva
  - [ ] Leitura das bolhas
  - [ ] Tela de resultado básica
- [ ] **v0.2 — Aprimoramentos**
  - [ ] Suporte a todos os 3 níveis
  - [ ] Leitura do código do aluno via QR Code (já presente no cartão)
  - [ ] Detecção robusta com iluminação variada
- [ ] **v1.0 — Release público**
  - [ ] Exportação de resultados (CSV)
  - [ ] Histórico de sessões
  - [ ] Testes automatizados com dataset de cartões reais

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Veja o arquivo [CONTRIBUTING.md](CONTRIBUTING.md) para saber como:

- Reportar bugs
- Sugerir novas funcionalidades
- Enviar Pull Requests

Por favor, leia também nosso [Código de Conduta](CODE_OF_CONDUCT.md).

---

## 📄 Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE) para mais informações.

---

## 👨‍💻 Autores

Projeto desenvolvido por **Eduardo Divino**, professor da rede estadual de ensino do estado de Goiás.

---

<p align="center">
  Feito com ❤️ para facilitar o trabalho de professores e ampliar o alcance da OBMEP
</p>
