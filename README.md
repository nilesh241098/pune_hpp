print(f"Selected Template: {selected_template}")
    print(f"Uploaded File: {uploaded_file.filename}, Content Type: {uploaded_file.content_type}")

    if not selected_template:
        return {"error": "No template selected!"}

    if not uploaded_file:
        return {"error": "No file uploaded!"}

    # Process the file as usual
    return {"response": "Debugging Successful"}
