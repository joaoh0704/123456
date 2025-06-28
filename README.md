node_modules
.expo
.envOPENAI_API_KEY=sua_chave_openai_aquimodule.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [['module:react-native-dotenv']],
  };
};import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, ScrollView, KeyboardAvoidingView, Platform } from 'react-native';
import { OPENAI_API_KEY } from '@env';

export default function App() {
  const [input, setInput] = useState('');
  const [chat, setChat] = useState([]);
  const [loading, setLoading] = useState(false);

  async function sendMessage() {
    if (!input.trim()) return;
    const userMessage = { from: 'user', text: input.trim() };
    setChat(prev => [...prev, userMessage]);
    setInput('');
    setLoading(true);

    try {
      const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${OPENAI_API_KEY}`,
        },
        body: JSON.stringify({
          model: 'gpt-4o',
          messages: [
            {
              role: 'system',
              content: 'Você é um assistente agronômico especializado que responde dúvidas técnicas, realiza cálculos de química agrícola, faz conversões de unidades e dá recomendações agrícolas detalhadas.',
            },
            {
              role: 'user',
              content: userMessage.text,
            },
          ],
        }),
      });

      const data = await response.json();
      if (data.choices && data.choices.length > 0) {
        const aiMessage = { from: 'ai', text: data.choices[0].message.content };
        setChat(prev => [...prev, aiMessage]);
      } else {
        setChat(prev => [...prev, { from: 'ai', text: 'Resposta vazia da IA.' }]);
      }
    } catch (e) {
      setChat(prev => [...prev, { from: 'ai', text: 'Erro ao conectar com a IA.' }]);
    }

    setLoading(false);
  }

  return (
    <KeyboardAvoidingView style={styles.container} behavior={Platform.OS === 'ios' ? 'padding' : undefined}>
      <ScrollView style={styles.chatArea} contentContainerStyle={{ paddingVertical: 10 }}>
        {chat.map((msg, i) => (
          <View key={i} style={[styles.message, msg.from === 'user' ? styles.userMsg : styles.aiMsg]}>
            <Text style={styles.messageText}>{msg.text}</Text>
          </View>
        ))}
      </ScrollView>

      <View style={styles.inputArea}>
        <TextInput
          style={styles.input}
          placeholder="Faça uma pergunta para a IA..."
          value={input}
          onChangeText={setInput}
          editable={!loading}
          onSubmitEditing={sendMessage}
        />
        <TouchableOpacity onPress={sendMessage} style={styles.sendButton} disabled={loading}>
          <Text style={styles.sendButtonText}>{loading ? '...' : 'Enviar'}</Text>
        </TouchableOpacity>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f2f9f1' },
  chatArea: { flex: 1, paddingHorizontal: 10 },
  message: {
    maxWidth: '80%',
    marginBottom: 10,
    padding: 10,
    borderRadius: 10,
  },
  userMsg: {
    backgroundColor: '#4caf50',
    alignSelf: 'flex-end',
    borderBottomRightRadius: 0,
  },
  aiMsg: {
    backgroundColor: '#dcedc8',
    alignSelf: 'flex-start',
    borderBottomLeftRadius: 0,
  },
  messageText: {
    color: '#000',
    fontSize: 16,
  },
  inputArea: {
    flexDirection: 'row',
    padding: 10,
    borderTopWidth: 1,
    borderColor: '#a5d6a7',
    backgroundColor: '#fff',
  },
  input: {
    flex: 1,
    backgroundColor: '#f0f4f1',
    borderRadius: 20,
    paddingHorizontal: 15,
    fontSize: 16,
  },
  sendButton: {
    marginLeft: 10,
    backgroundColor: '#4caf50',
    borderRadius: 20,
    paddingVertical: 10,
    paddingHorizontal: 20,
    justifyContent: 'center',
  },
  sendButtonText: {
    color: '#fff',
    fontWeight: '600',
    fontSize: 16,
  },
});
