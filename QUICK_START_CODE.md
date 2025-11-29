# PlacementPanic - Quick Start Code Guide

## Quick Installation

```bash
# 1. Install all required packages
npm install axios chart.js react-chartjs-2 date-fns uuid @react-pdf/renderer pdfmake ml5 @xenova/transformers

# 2. Create these folders
mkdir -p client/src/components client/src/hooks client/src/lib

# 3. Copy files from this guide into respective folders
```

---

## Step 1: Create Hook File - `client/src/hooks/useVoiceRecorder.ts`

Copy the entire content from AI_FEATURES_IMPLEMENTATION.md section "1. Voice Recording Hook"

---

## Step 2: Create Hook File - `client/src/hooks/useFacialAnalysis.ts`

Copy the entire content from AI_FEATURES_IMPLEMENTATION.md section "2. Facial Analysis Hook"

---

## Step 3: Create Utility - `client/src/lib/confidenceCalculator.ts`

Copy the entire content from AI_FEATURES_IMPLEMENTATION.md section "3. Confidence Calculator"

---

## Step 4: Create Service - `client/src/lib/answerEvaluator.ts`

Copy the entire content from AI_FEATURES_IMPLEMENTATION.md section "4. Answer Evaluator"

---

## Step 5: Update Backend - Add to `server/routes.ts`

```typescript
// Add these imports at the top
import express from 'express';
import { authenticateToken } from './middleware';  // Adjust path if needed

// Add these routes before export default app

// Transcription endpoint
app.post('/api/transcribe', authenticateToken, async (req, res) => {
  try {
    const { audio } = req.body;
    // Use AssemblyAI or OpenAI Whisper
    // For now, return mock transcript
    res.json({ transcript: 'Mock transcript of audio' });
  } catch (error) {
    res.status(500).json({ error: 'Transcription failed' });
  }
});

// Answer evaluation endpoint
app.post('/api/evaluate-answer', authenticateToken, async (req, res) => {
  try {
    const { question, userAnswer } = req.body;
    // Integrate with OpenAI API
    const evaluation = {
      correctness: 85,
      completeness: 78,
      clarity: 82,
      keyPoints: ['Key point 1', 'Key point 2'],
      missingPoints: ['Missing 1'],
      suggestions: ['Suggestion 1']
    };
    res.json({ evaluation });
  } catch (error) {
    res.status(500).json({ error: 'Evaluation failed' });
  }
});

// Facial analysis endpoint
app.post('/api/analyze-facial', authenticateToken, async (req, res) => {
  try {
    const { frameData } = req.body;
    // Process facial metrics
    res.json({
      confidence: 85,
      emotions: {
        happy: 45,
        neutral: 30,
        sad: 10,
        angry: 5,
        fearful: 5,
        surprised: 5
      },
      eyeContact: 80,
      headPosition: 'center'
    });
  } catch (error) {
    res.status(500).json({ error: 'Analysis failed' });
  }
});
```

---

## Step 6: Create Component - `client/src/components/VoiceRecorder.tsx`

Copy from AI_FEATURES_IMPLEMENTATION.md section "6. VoiceRecorder Component"

---

## Step 7: Create Component - `client/src/components/ConfidenceScoreCard.tsx`

```typescript
import React from 'react';
import { TrendingUp } from 'lucide-react';

interface ConfidenceScoreCardProps {
  score: number;
  level: string;
  breakdown?: Record<string, number>;
}

export function ConfidenceScoreCard({ score, level, breakdown }: ConfidenceScoreCardProps) {
  const getColorClass = () => {
    if (score >= 80) return 'bg-green-500';
    if (score >= 65) return 'bg-blue-500';
    if (score >= 50) return 'bg-yellow-500';
    return 'bg-red-500';
  };

  return (
    <div className={`p-6 rounded-lg text-white ${getColorClass()}`}>
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-lg font-semibold">Confidence Score</h3>
        <TrendingUp className="w-5 h-5" />
      </div>
      
      <div className="text-4xl font-bold mb-2">{score}%</div>
      <p className="text-sm opacity-90 mb-4">{level}</p>
      
      {breakdown && (
        <div className="space-y-2 text-sm">
          <div className="flex justify-between">
            <span>Voice Quality:</span>
            <span>{breakdown.voiceStability || 0}%</span>
          </div>
          <div className="flex justify-between">
            <span>Facial Expression:</span>
            <span>{breakdown.facialConfidence || 0}%</span>
          </div>
          <div className="flex justify-between">
            <span>Answer Quality:</span>
            <span>{breakdown.answerQuality || 0}%</span>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Step 8: Update .env File

```
VITE_OPENAI_API_KEY=sk_test_your_key_here
VITE_GOOGLE_VISION_API_KEY=your_google_vision_key
VITE_ASSEMBLYAI_API_KEY=your_assemblyai_key
```

---

## Step 9: Test Everything

```bash
# Start development server
npm run dev

# Check console for any errors
# Test voice recording - should show transcript
# Test facial analysis - should show emotions
# Test confidence scoring - should calculate based on metrics
```

---

## Integration with Interview Component

Update your interview page like this:

```typescript
import { VoiceRecorder } from '../components/VoiceRecorder';
import { ConfidenceScoreCard } from '../components/ConfidenceScoreCard';
import { useVoiceRecorder } from '../hooks/useVoiceRecorder';
import { calculateConfidenceScore } from '../lib/confidenceCalculator';

export function InterviewPage() {
  const { transcript, isRecording } = useVoiceRecorder();
  const [confidenceScore, setConfidenceScore] = useState(0);

  const handleAnswerSubmit = (answer: string) => {
    const confidenceData = {
      voiceVolume: 75,
      voiceStability: 80,
      pauseFrequency: 20,
      facialConfidence: 85,
      emotionPositivity: 40,
      answerQuality: 78,
      answerCompleteness: 82
    };
    
    const { score, level } = calculateConfidenceScore(confidenceData);
    setConfidenceScore(score);
  };

  return (
    <div className="grid grid-cols-3 gap-6">
      <div className="col-span-2">
        <VoiceRecorder />
        <button 
          onClick={() => handleAnswerSubmit(transcript)}
          className="mt-4 bg-blue-500 text-white px-6 py-2 rounded-lg"
        >
          Analyze Answer
        </button>
      </div>
      <aside>
        <ConfidenceScoreCard score={confidenceScore} level="Confident" />
      </aside>
    </div>
  );
}
```

---

## Troubleshooting

### Microphone not working?
- Check browser permissions
- Ensure using HTTPS on production
- Check MediaRecorder browser support

### API errors?
- Verify API keys in .env
- Check network tab in DevTools
- Ensure backend is running on correct port

### Facial detection not working?
- Load ml5.js from CDN in index.html
- Add: `<script src="https://unpkg.com/ml5@latest/dist/ml5.min.js"></script>`
- Check webcam permissions

---

## Next: Advanced Features

1. Resume parser with PDF extraction
2. Multi-round conversational interviews
3. Adaptive difficulty based on performance
4. Detailed analytics dashboard
5. PDF report generation
6. Email notifications
7. Leaderboards and peer comparison

**See AI_FEATURES_IMPLEMENTATION.md for complete details!**
