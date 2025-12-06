Ús de SpeechRecognizer (API nativa, sense TTS)

Android ja porta el reconeixedor integrat i no cal cap SDK extern.

Passos:

### 1. Permís al manifest
<uses-permission android:name="android.permission.RECORD_AUDIO" />

### 2. Crear el reconeixedor
private lateinit var recognizer: SpeechRecognizer

### 3. Inicialitzar-lo
```kotlin
recognizer = SpeechRecognizer.createSpeechRecognizer(this)

val recognizerIntent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
    putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
    putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ca-ES") // o "es-ES"
}
```

### 4. Assignar un listener
```kotlin
recognizer.setRecognitionListener(object : RecognitionListener {
    override fun onResults(results: Bundle?) {
        val spokenText = results
            ?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
            ?.get(0)
            ?.lowercase()

        handleVoiceCommand(spokenText)
    }

    override fun onError(error: Int) {}
    override fun onReadyForSpeech(params: Bundle?) {}
    override fun onBeginningOfSpeech() {}
    override fun onRmsChanged(rmsdB: Float) {}
    override fun onBufferReceived(buffer: ByteArray?) {}
    override fun onEndOfSpeech() {}
    override fun onPartialResults(partialResults: Bundle?) {}
    override fun onEvent(eventType: Int, params: Bundle?) {}
})
```

### 5. Interpretar les ordres
```kotlin
private fun handleVoiceCommand(command: String?) {
    when {
        command?.contains("endavant") == true -> {
            // Obrir menú, navegar, etc.
        }
        command?.contains(" enrere") == true -> {
            onBackPressedDispatcher.onBackPressed()
        }
        command?.contains("acceptar") == true -> {
            // Guardar dades, fer submit, etc.
        }
    }
}
```

### 6. Per activar el reconeixedor quan l’usuari parla

alguna cosa com:
```kotlin
buttonVoice.setOnClickListener {
    recognizer.startListening(recognizerIntent)
}
```
