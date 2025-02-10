# frozone-suppurts
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const multer = require("multer");
const { Translate } = require("@google-cloud/translate").v2;
const SpeechToText = require("@google-cloud/speech");
require("dotenv").config();

const app = express();
const port = process.env.PORT || 5000;
const translate = new Translate();
const speechClient = new SpeechToText.SpeechClient();

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB connected"))
.catch(err => console.log(err));

// Image Upload (Using Multer)
const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

// Language Translation API
app.post("/translate", async (req, res) => {
  try {
    const { text, targetLang } = req.body;
    let [translation] = await translate.translate(text, targetLang);
    res.json({ translatedText: translation });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Speech-to-Text API
app.post("/speech", async (req, res) => {
  try {
    const audio = req.body.audio;
    const [response] = await speechClient.recognize({
      config: { encoding: "LINEAR16", languageCode: "en-US" },
      audio: { content: audio },
    });
    res.json({ transcript: response.results[0].alternatives[0].transcript });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Basic Route
app.get("/", (req, res) => {
  res.send("Chatbot Backend is Running");
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

