# Google AI Python SDK Example

This project demonstrates how to use the Google AI Python SDK to upload files to Gemini and interact with the model.

## Getting Started

Follow these instructions to set up and run the project.

### Prerequisites

Make sure you have Python installed. You can download it from [python.org](https://www.python.org/).

### Installation

1. Install the Google AI Python SDK:

    ```sh
    pip install google-generativeai
    ```

2. Set up your Google AI API key:

    ```python
    import google.generativeai as genai

    genai.configure(api_key="YOUR_API_KEY")
    ```

### Usage

1. Define the function to upload a file to Gemini:

    ```python
    import os
    import time
    import google.generativeai as genai

    genai.configure(api_key="YOUR_API_KEY")

    def upload_to_gemini(path, mime_type=None):
        file = genai.upload_file(path, mime_type=mime_type)
        print(f"Uploaded file '{file.display_name}' as: {file.uri}")
        return file
    ```

2. Define the function to wait for files to become active:

    ```python
    def wait_for_files_active(files):
        print("Waiting for file processing...")
        for name in (file.name for file in files):
            file = genai.get_file(name)
            while file.state.name == "PROCESSING":
                print(".", end="", flush=True)
                time.sleep(10)
                file = genai.get_file(name)
            if file.state.name != "ACTIVE":
                raise Exception(f"File {file.name} failed to process")
        print("...all files ready")
        print()
    ```

3. Configure the model:

    ```python
    generation_config = {
        "temperature": 1,
        "top_p": 0.95,
        "top_k": 64,
        "max_output_tokens": 8192,
        "response_mime_type": "text/plain",
    }
    safety_settings = [
        {
            "category": "HARM_CATEGORY_HARASSMENT",
            "threshold": "BLOCK_MEDIUM_AND_ABOVE",
        },
        {
            "category": "HARM_CATEGORY_HATE_SPEECH",
            "threshold": "BLOCK_MEDIUM_AND_ABOVE",
        },
        {
            "category": "HARM_CATEGORY_SEXUALLY_EXPLICIT",
            "threshold": "BLOCK_MEDIUM_AND_ABOVE",
        },
        {
            "category": "HARM_CATEGORY_DANGEROUS_CONTENT",
            "threshold": "BLOCK_MEDIUM_AND_ABOVE",
        },
    ]

    model = genai.GenerativeModel(
        model_name="gemini-1.5-flash",
        safety_settings=safety_settings,
        generation_config=generation_config,
    )
    ```

4. Upload a video file and wait for it to become active:

    ```python
    video_path = input("Enter Your Video Path: ")
    files = [upload_to_gemini(video_path, mime_type="video/mp4")]
    wait_for_files_active(files)
    ```

5. Start a chat session and interact with the model:

    ```python
    chat_session = model.start_chat(
        history=[
            {
                "role": "user",
                "parts": [
                    files[0],
                ],
            },
        ]
    )
    prompt = input("Please Ask A Question About Video: ")
    response = chat_session.send_message(prompt)

    print(response.text)
    ```

### Contributing

If you would like to contribute to this project, please fork the repository and create a pull request.

### License

This project is licensed under the MIT License. See the `LICENSE` file for details.

### References

- [Google AI Python SDK Documentation](https://ai.google.dev/gemini-api/docs/get-started/python)
- [Prompting with Media](https://ai.google.dev/gemini-api/docs/prompting_with_media)

