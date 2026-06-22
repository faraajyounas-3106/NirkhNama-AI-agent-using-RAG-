# Nirkh Nama

A multi-agent AI system designed to protect consumers in Pakistan from overcharging by shopkeepers. Nirkh Nama cross-references the official District Commission daily price list against receipts submitted via image scan or voice recording, and flags any item that is sold above the government-approved rate.

---

## What It Does

Shopkeepers in Pakistan are legally required to sell goods at rates set by the District Price Control Committee. These rates are published daily as a PDF, known as the Nirkh Nama. In practice, consumers rarely have access to this document at the point of sale and cannot verify whether they are being overcharged.

This system solves that problem by:

- Accepting a receipt as a scanned image or a voice recording
- Extracting item names, quantities, and prices from the input
- Retrieving the current approved rate for each item from the official PDF using RAG
- Comparing the charged price against the approved rate
- Reporting any discrepancy clearly to the user

---

## Architecture

The system is composed of several specialized agents working in a pipeline.

### Input Layer

**Voice Agent (Whisper Flow API)**
Converts spoken input into text. A user can verbally describe what they bought and at what price, and the agent transcribes it accurately, handling regional accents and mixed-language input common in Pakistani markets.

**Image Agent (Gemini API)**
Accepts a photograph of a receipt or handwritten bill. Gemini extracts all visible text from the image, including item names, units, quantities, and prices, even from low-quality or handwritten receipts.

### Processing Layer

**Text-to-Token Agent (OpenAI JSON Agent)**
Takes the raw text output from either the voice or image agent and structures it into a standardized JSON format. Each line item is converted into a token containing the item name, quantity, unit, and price charged. This normalization step ensures the downstream RAG query is consistent regardless of input source.

Example output:

```json
{
  "items": [
    { "name": "Tomato", "quantity": 1, "unit": "kg", "price_charged": 180 },
    { "name": "Onion", "quantity": 2, "unit": "kg", "price_charged": 160 }
  ]
}
```

### Knowledge Layer

**RAG Agent (PDF Retrieval)**
The official District Commission daily price list PDF is ingested, chunked, embedded, and stored in a vector store. When item tokens arrive from the processing layer, this agent queries the store for the current approved rate of each item. The PDF is refreshed daily to ensure the rates are always up to date.

### Output Layer

**Comparison and Report Agent**
Compares the price charged for each item against the retrieved approved rate. Calculates the overcharge amount and percentage. Generates a plain-language report for the user indicating which items were fairly priced and which were sold above the legal limit.

---

## Project Structure

```
.
├── app.py                  # Main application entry point and agent orchestration
├── daily_price_list.pdf    # Official District Commission price list (refreshed daily)
├── requirements.txt        # Python dependencies
├── Dockerfile              # Container configuration
├── .env.example            # Environment variable template
├── .dockerignore
├── .gitignore
└── README.md
```

---

## Setup

### Prerequisites

- Python 3.10 or higher
- Docker (optional, for containerized deployment)
- API keys for OpenAI, Google Gemini, and Whisper Flow

### Installation

Clone the repository:

```bash
git clone https://github.com/your-username/nirkh-nama.git
cd nirkh-nama
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Copy the environment variable template and fill in your API keys:

```bash
cp .env.example .env
```

Open `.env` and set the following variables:

```
OPENAI_API_KEY=your_openai_key
GEMINI_API_KEY=your_gemini_key
WHISPER_FLOW_API_KEY=your_whisper_flow_key
PDF_PATH=daily_price_list.pdf
```

### Running the Application

```bash
python app.py
```

### Running with Docker

```bash
docker build -t nirkh-nama .
docker run --env-file .env nirkh-nama
```

---

## Usage

### Scan a Receipt

Provide an image of a receipt (JPG or PNG). The image agent will extract all line items automatically.

```bash
python app.py --mode image --input receipt.jpg
```

### Record a Voice Description

Speak your purchase aloud. The voice agent will transcribe and parse your input.

```bash
python app.py --mode voice --input recording.mp3
```

### Sample Output

```
Nirkh Nama Report
Date: 2025-06-22
District: Rawalpindi

Item          Qty    Charged    Approved Rate    Status
-------------------------------------------------------------
Tomato        1 kg   Rs. 180    Rs. 140          OVERCHARGED by Rs. 40 (28.5%)
Onion         2 kg   Rs. 160    Rs. 160          FAIR
Potato        1 kg   Rs. 95     Rs. 80           OVERCHARGED by Rs. 15 (18.75%)

Summary: 2 out of 3 items were charged above the approved rate.
Total overcharge: Rs. 55
```

---

## Updating the Price List

The `daily_price_list.pdf` must be replaced with the latest version from the District Price Control Committee each day. The RAG index is rebuilt automatically on application startup if the PDF has been updated.

---

## Dependencies

Key libraries and APIs used in this project:

- Whisper Flow API for speech-to-text transcription
- Google Gemini API for image-based receipt extraction
- OpenAI API for JSON-structured item tokenization
- A vector database for RAG-based PDF retrieval
- LangChain or equivalent for agent orchestration

See `requirements.txt` for the full list of Python packages.

---

## Contributing

Pull requests are welcome. For significant changes, please open an issue first to discuss what you would like to change.

---

## License

See the LICENSE file for details.

---

## Disclaimer

This tool is intended to help consumers verify prices against the official government-approved rates. It does not constitute legal advice. Rate data is only as accurate as the PDF published by the District Price Control Committee. Always verify against the latest official source.
