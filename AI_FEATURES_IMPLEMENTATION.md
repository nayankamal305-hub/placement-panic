# PlacementPanic - AI Enhanced Features Implementation Guide

## Overview
This guide provides complete implementation details for adding AI-powered features to PlacementPanic including voice recording, facial expression analysis, confidence scoring, and answer evaluation.

## New Features Added

### 1. Voice Recording & Speech-to-Text (STT)
### 2. Webcam & Facial Expression Analysis  
### 3. Confidence Scoring Engine
### 4. AI Answer Evaluation
### 5. Analytics Dashboard with Performance Charts
### 6. Conversational Follow-up Questions
### 7. Resume Parser
### 8. Adaptive Difficulty System
### 9. Detailed Report Generation

---

## Installation & Setup

### Step 1: Install New Dependencies

```bash
npm install
  axios
  chart.js
  react-chartjs-2
  date-fns
  uuid
  @react-pdf/renderer
  pdfmake
  ml5
  @xenova/transformers
```

### Step 2: Environment Variables

Add to your `.env` file:

```
VITE_OPENAI_API_KEY=your_openai_api_key
VITE_GOOGLE_VISION_API_KEY=your_google_vision_api_key  
VITE_ASSEMBLYAI_API_KEY=your_assemblyai_api_key
```

---

## File Structure

Create the following new files in your project:

```
placement-panic/
  client/src/
    components/
      VoiceRecorder.tsx          # Voice recording component
      FacialAnalyzer.tsx         # Facial expression detection
      ConfidenceScoreCard.tsx    # Confidence visualization
      AnswerEvaluator.tsx        # AI answer evaluation
      PerformanceChart.tsx       # Charts dashboard
      ResumeUploader.tsx         # Resume parser
      DetailedReport.tsx         # Report generation
    lib/
      aiServices.ts             # AI API integrations
      confidenceCalculator.ts    # Confidence scoring logic
      answerEvaluator.ts        # Answer evaluation logic
      facialAnalysis.ts         # Facial emotion detection
      resumeParser.ts           # Resume parsing logic
      reportGenerator.ts        # Report generation logic
    hooks/
      useVoiceRecorder.ts       # Voice recording hook
      useFacialAnalysis.ts      # Facial analysis hook
      useConfidenceScore.ts     # Confidence score hook
```

---
## Implementation Details

### 1. Voice Recording Hook (useVoiceRecorder.ts)

```typescript
import { useState, useRef } from 'react';

interface UseVoiceRecorderReturn {
  isRecording: boolean;
  audioBlob: Blob | null;
  startRecording: () => void;
  stopRecording: () => void;
  transcript: string;
}

export function useVoiceRecorder(): UseVoiceRecorderReturn {
  const [isRecording, setIsRecording] = useState(false);
  const [audioBlob, setAudioBlob] = useState<Blob | null>(null);
  const [transcript, setTranscript] = useState('');
  const mediaRecorderRef = useRef<MediaRecorder | null>(null);
  const chunksRef = useRef<Blob[]>([]);

  const startRecording = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const mediaRecorder = new MediaRecorder(stream);
      mediaRecorderRef.current = mediaRecorder;
      chunksRef.current = [];

      mediaRecorder.ondataavailable = (event) => {
        if (event.data.size > 0) {
          chunksRef.current.push(event.data);
        }
      };

      mediaRecorder.onstop = async () => {
        const blob = new Blob(chunksRef.current, { type: 'audio/webm' });
        setAudioBlob(blob);
        
        // Send to STT service
        const text = await transcribeAudio(blob);
        setTranscript(text);
      };

      mediaRecorder.start();
      setIsRecording(true);
    } catch (error) {
      console.error('Error accessing microphone:', error);
    }
  };

  const stopRecording = () => {
    if (mediaRecorderRef.current && isRecording) {
      mediaRecorderRef.current.stop();
      mediaRecorderRef.current.stream.getTracks().forEach(track => track.stop());
      setIsRecording(false);
    }
  };

  return { isRecording, audioBlob, startRecording, stopRecording, transcript };
}

async function transcribeAudio(audioBlob: Blob): Promise<string> {
  const formData = new FormData();
  formData.append('audio', audioBlob);

  try {
    const response = await fetch('/api/transcribe', {
      method: 'POST',
      body: formData,
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    });
    const data = await response.json();
    return data.transcript;
  } catch (error) {
    console.error('Transcription error:', error);
    return '';
  }
}
```

### 2. Facial Analysis Hook (useFacialAnalysis.ts)

```typescript
import { useState, useRef, useEffect } from 'react';

interface FacialMetrics {
  confidence: number;
  emotions: {
    happy: number;
    neutral: number;
    sad: number;
    angry: number;
    fearful: number;
    surprised: number;
  };
  eyeContact: number;
  headPosition: string;
}

export function useFacialAnalysis() {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isAnalyzing, setIsAnalyzing] = useState(false);
  const [facialMetrics, setFacialMetrics] = useState<FacialMetrics | null>(null);
  const [cumulativeScore, setCumulativeScore] = useState(0);
  const metricsRef = useRef<FacialMetrics[]>([]);

  const startAnalysis = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        setIsAnalyzing(true);
        analyzeFacialExpression();
      }
    } catch (error) {
      console.error('Error accessing camera:', error);
    }
  };

  const analyzeFacialExpression = async () => {
    // Using ml5.js for emotion detection
    const ml5 = (window as any).ml5;
    if (!ml5) return;

    const detection = ml5.faceApi(
      videoRef.current,
      () => {
        console.log('ml5.js ready');
      }
    );

    const analyzeFrame = () => {
      if (isAnalyzing && videoRef.current) {
        const detections = detection.detections;
        
        if (detections && detections.length > 0) {
          const face = detections[0];
          const expressions = face.expressions;
          const landmarks = face.landmarks;

          const metrics: FacialMetrics = {
            confidence: face.detection.score,
            emotions: expressions,
            eyeContact: calculateEyeContact(landmarks),
            headPosition: getHeadPosition(landmarks)
          };

          setFacialMetrics(metrics);
          metricsRef.current.push(metrics);
        }
        requestAnimationFrame(analyzeFrame);
      }
    };

    analyzeFrame();
  };

  const stopAnalysis = () => {
    setIsAnalyzing(false);
    if (videoRef.current && videoRef.current.srcObject) {
      (videoRef.current.srcObject as MediaStream)
        .getTracks()
        .forEach(track => track.stop());
    }
    
    // Calculate cumulative score
    const avgScore = calculateAverageConfidence(metricsRef.current);
    setCumulativeScore(avgScore);
  };

  return {
    videoRef,
    isAnalyzing,
    startAnalysis,
    stopAnalysis,
    facialMetrics,
    cumulativeScore
  };
}

function calculateEyeContact(landmarks: any): number {
  // Calculate if eyes are looking at camera (0-100)
  return 85; // Placeholder
}

function getHeadPosition(landmarks: any): string {
  return 'center'; // Placeholder
}

function calculateAverageConfidence(metrics: FacialMetrics[]): number {
  if (metrics.length === 0) return 0;
  const total = metrics.reduce((sum, m) => sum + m.confidence, 0);
  return (total / metrics.length) * 100;
}
```

### 3. Confidence Calculator (confidenceCalculator.ts)

```typescript
interface ConfidenceInputs {
  voiceVolume: number;           // 0-100
  voiceStability: number;        // 0-100
  pauseFrequency: number;        // 0-100 (lower is better)
  facialConfidence: number;      // 0-100
  emotionPositivity: number;     // -100 to 100
  answerQuality: number;         // 0-100
  answerCompleteness: number;    // 0-100
}

export function calculateConfidenceScore(inputs: ConfidenceInputs): {
  score: number;
  level: string;
  breakdown: Record<string, number>;
} {
  const weights = {
    voiceVolume: 0.15,
    voiceStability: 0.15,
    pauseFrequency: 0.1,
    facialConfidence: 0.2,
    emotionPositivity: 0.1,
    answerQuality: 0.15,
    answerCompleteness: 0.15
  };

  const normalizedInputs = {
    voiceVolume: inputs.voiceVolume,
    voiceStability: inputs.voiceStability,
    pauseFrequency: 100 - inputs.pauseFrequency, // Invert
    facialConfidence: inputs.facialConfidence,
    emotionPositivity: inputs.emotionPositivity + 100 / 2, // Normalize to 0-100
    answerQuality: inputs.answerQuality,
    answerCompleteness: inputs.answerCompleteness
  };

  const score =
    normalizedInputs.voiceVolume * weights.voiceVolume +
    normalizedInputs.voiceStability * weights.voiceStability +
    normalizedInputs.pauseFrequency * weights.pauseFrequency +
    normalizedInputs.facialConfidence * weights.facialConfidence +
    normalizedInputs.emotionPositivity * weights.emotionPositivity +
    normalizedInputs.answerQuality * weights.answerQuality +
    normalizedInputs.answerCompleteness * weights.answerCompleteness;

  const level = getConfidenceLevel(score);
  const breakdown = normalizedInputs;

  return { score: Math.round(score), level, breakdown };
}

function getConfidenceLevel(score: number): string {
  if (score >= 80) return 'Very Confident';
  if (score >= 65) return 'Confident';
  if (score >= 50) return 'Moderately Confident';
  if (score >= 35) return 'Less Confident';
  return 'Needs Improvement';
}
```

### 4. Answer Evaluator (answerEvaluator.ts)

```typescript
import axios from 'axios';

interface AnswerEvaluation {
  correctness: number;        // 0-100
  completeness: number;       // 0-100
  clarity: number;            // 0-100
  keyPointsCovered: string[];
  missingPoints: string[];
  suggestions: string[];
  overallScore: number;       // 0-100
}

export async function evaluateAnswer(
  question: string,
  userAnswer: string,
  expectedAnswer?: string
): Promise<AnswerEvaluation> {
  try {
    const response = await axios.post('/api/evaluate-answer', {
      question,
      userAnswer,
      expectedAnswer
    }, {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      }
    });

    return response.data.evaluation;
  } catch (error) {
    console.error('Answer evaluation error:', error);
    throw error;
  }
}

export function parseEvaluationResponse(response: any): AnswerEvaluation {
  return {
    correctness: response.correctness || 0,
    completeness: response.completeness || 0,
    clarity: response.clarity || 0,
    keyPointsCovered: response.keyPoints || [],
    missingPoints: response.missingPoints || [],
    suggestions: response.suggestions || [],
    overallScore: Math.round(
      (response.correctness + response.completeness + response.clarity) / 3
    )
  };
}
```

### 5. Backend API Endpoint for Answer Evaluation

Add to `server/routes.ts`:

```typescript
app.post('/api/evaluate-answer', authenticateToken, async (req, res) => {
  const { question, userAnswer, expectedAnswer } = req.body;

  try {
    const evaluation = await evaluateWithOpenAI({
      question,
      userAnswer,
      expectedAnswer
    });

    res.json({ evaluation });
  } catch (error) {
    res.status(500).json({ error: 'Evaluation failed' });
  }
});

async function evaluateWithOpenAI(data: any) {
  const openaiApiKey = process.env.VITE_OPENAI_API_KEY;
  
  const prompt = `
Evaluate this interview answer:

Question: ${data.question}
User's Answer: ${data.userAnswer}
${data.expectedAnswer ? `Expected/Ideal Answer: ${data.expectedAnswer}` : ''}

Provide evaluation in JSON format:
{
  "correctness": <0-100>,
  "completeness": <0-100>,
  "clarity": <0-100>,
  "keyPoints": ["point1", "point2"],
  "missingPoints": ["missing1"],
  "suggestions": ["suggestion1"]
}
`;

  // Call OpenAI API
  // Parse and return response
}
```

---

## Database Schema Updates

Add new tables to `server/storage.ts`:

```typescript
interface InterviewSession {
  id: string;
  userId: string;
  startTime: Date;
  endTime?: Date;
  totalConfidenceScore: number;
  facialExpressionScore: number;
  voiceAnalysisScore: number;
  answerQualityScore: number;
  questions: QuestionWithResponse[];
  resumeId?: string;
  interviewMode: 'standard' | 'with_video' | 'with_voice' | 'full_ai';
}

interface QuestionWithResponse {
  questionId: string;
  question: string;
  userAnswer: string;
  voiceTranscript?: string;
  videoUri?: string;
  facialMetrics?: any;
  evaluation?: AnswerEvaluation;
  confidence: number;
  timestamp: Date;
  followUpQuestions?: string[];
}

interface ResumeData {
  id: string;
  userId: string;
  fileName: string;
  parsedData: {
    name: string;
    email: string;
    phone: string;
    skills: string[];
    experience: any[];
    education: any[];
    projects: any[];
  };
  uploadedAt: Date;
}
```

---

## Frontend Components

### 6. VoiceRecorder Component

```typescript
import React from 'react';
import { useVoiceRecorder } from '../hooks/useVoiceRecorder';
import { Mic, StopCircle } from 'lucide-react';

export function VoiceRecorder() {
  const { isRecording, startRecording, stopRecording, transcript } = useVoiceRecorder();

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        {!isRecording ? (
          <button
            onClick={startRecording}
            className="flex items-center gap-2 bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
          >
            <Mic className="w-4 h-4" />
            Start Recording
          </button>
        ) : (
          <button
            onClick={stopRecording}
            className="flex items-center gap-2 bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600"
          >
            <StopCircle className="w-4 h-4" />
            Stop Recording
          </button>
        )}
      </div>
      {transcript && (
        <div className="p-4 bg-gray-100 rounded-lg">
          <p className="text-sm font-medium text-gray-700">Transcript:</p>
          <p className="text-gray-900 mt-2">{transcript}</p>
        </div>
      )}
    </div>
  );
}
```

---

## Integration with Existing Interview Component

Update your existing interview page to use the new features:

```typescript
import { VoiceRecorder } from '../components/VoiceRecorder';
import { FacialAnalyzer } from '../components/FacialAnalyzer';
import { ConfidenceScoreCard } from '../components/ConfidenceScoreCard';
import { AnswerEvaluator } from '../components/AnswerEvaluator';

export function EnhancedInterviewPage() {
  const [useVoice, setUseVoice] = useState(false);
  const [useCamera, setUseCamera] = useState(false);
  const [confidenceScore, setConfidenceScore] = useState(0);

  return (
    <div className="space-y-6">
      {/* Feature toggles */}
      <div className="flex gap-4">
        <label className="flex items-center gap-2">
          <input
            type="checkbox"
            checked={useVoice}
            onChange={(e) => setUseVoice(e.target.checked)}
          />
          <span>Record Voice</span>
        </label>
        <label className="flex items-center gap-2">
          <input
            type="checkbox"
            checked={useCamera}
            onChange={(e) => setUseCamera(e.target.checked)}
          />
          <span>Enable Webcam</span>
        </label>
      </div>

      {/* Interview content */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2 space-y-4">
          {useVoice && <VoiceRecorder />}
          {useCamera && <FacialAnalyzer />}
          <AnswerEvaluator />
        </div>
        <aside>
          <ConfidenceScoreCard score={confidenceScore} />
        </aside>
      </div>
    </div>
  );
}
```

---

## Setup Instructions

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Add environment variables to `.env`:**
   ```
   VITE_OPENAI_API_KEY=your_key
   VITE_GOOGLE_VISION_API_KEY=your_key
   ```

3. **Copy component files to `client/src/components/`**

4. **Copy hook files to `client/src/hooks/`**

5. **Update backend routes in `server/routes.ts`**

6. **Start development server:**
   ```bash
   npm run dev
   ```

---

## API Endpoints to Add

```
POST /api/transcribe              - Convert audio to text
POST /api/evaluate-answer         - Evaluate user's answer
POST /api/generate-followup       - Generate follow-up questions
POST /api/analyze-facial          - Analyze facial metrics
POST /api/parse-resume            - Parse uploaded resume
GET  /api/interviews/:id/report   - Get detailed report
POST /api/adaptive-difficulty     - Get adjusted difficulty
```

---

## Next Steps

1. Create all component files with provided code
2. Set up API endpoints in backend
3. Add database models for new features
4. Test voice recording and transcription
5. Integrate facial analysis
6. Deploy to production

**For detailed component code, see the accompanying files in this repo.**
