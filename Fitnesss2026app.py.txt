import streamlit as st
import google.generativeai as genai

# 1. Set up the App Interface Layout
st.set_page_config(page_title="GreenFit AI", page_icon="🥗")
st.title("🥗 GreenFit: 100% Vegetarian AI Fitness Coach")
st.subheader("Talk to your AI coach about your workouts, diet, or missing ingredients!")

# 2. Securely Ask for the User's Google API Key
user_api_key = st.text_input("🔑 Enter your Google Gemini API Key to activate the AI:", type="password")

if user_api_key:
    # --- FIX: Securely configure the Gemini SDK with the provided key ---
    genai.configure(api_key=user_api_key)
    
    # Initialize the model (using the current standard Gemini model)
    model = genai.GenerativeModel("gemini-1.5-flash")
    
    # System instructions to enforce the 100% vegetarian persona
    system_prompt = (
        "You are GreenFit, an expert fitness coach and nutritionist who specializes strictly "
        "in 100% vegetarian diets. Provide workout routines, high-protein vegetarian meal plans, "
        "and suggestions for replacing missing ingredients with vegetarian alternatives. Never suggest "
        "meat, fish, or poultry."
    )

    # 3. Chat Interface Setup
    if "messages" not in st.session_state:
        st.session_state.messages = []

    # Display existing chat history
    for message in st.session_state.messages:
        with st.chat_message(message["role"]):
            st.markdown(message["content"])

    # Accept user input
    if user_input := st.chat_input("Ask your vegetarian fitness coach..."):
        # Display user message
        with st.chat_message("user"):
            st.markdown(user_input)
        st.session_state.messages.append({"role": "user", "content": user_input})

        # Generate AI response
        with st.chat_message("assistant"):
            response_placeholder = st.empty()
            
            try:
                # Combine system prompt with history for context
                full_prompt = f"{system_prompt}\n\nUser: {user_input}"
                response = model.generate_content(full_prompt)
                
                ai_response = response.text
                response_placeholder.markdown(ai_response)
                st.session_state.messages.append({"role": "assistant", "content": ai_response})
                
            except Exception as e:
                error_message = f"⚠️ An error occurred: {str(e)}\n\nPlease check if your API key is valid and active."
                response_placeholder.error(error_message)

else:
    st.info("Please enter your Gemini API key above to start chatting with your coach.")
