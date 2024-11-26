# Telegram-Bot-Powered-by-AI-for-psychological-Issues
Kira is a Telegram bot powered by artificial intelligence, designed to help users with psychological issues, including relationship contexts. Additionally, Kira offers thematic psychological tests to users. There is also Kira+, which differs from the basic Kira in that Kira is free with a limited set of tests, while Kira+ provides full functionality





Kira
Kira +
Goal
Engagement
Monetization
Price
Free
N USD per month for subscription
Functionality
AI consultations on the subject + free relationship tests + advertisements promoting Kira+ features.”
AI consultations + relationship tests + tests with visualizations + sexology tests.






The psychological consultation workflow can be observed https://t.me/pocket_psychologyst_bot.
The project will be built using the source code from this bot, eliminating the need to start from scratch. The bot is based on aiogram, with PostgreSQL as the database 


Candidate Requirements
Proven experience in developing Telegram bots using aiogram.
Basic experience with Docker.
Experience integrating payment systems into Telegram bots, requiring recurring payment setups.

Functional Requirements
Kira provides AI-based psychological consultations related to relationships. The implementation should be based on chatGPT, with easy switching to gigaChat or YaGPT (already implemented).
The main part of the prompt is static, but it can be supplemented with user information like age and other significant facts, which will be collected during interaction.
Kira and Kira+ use context from the last N messages along with “significant user facts,” such as test results that could affect the consultation's outcome.
User entity characterization includes:
Year of Birth (used to calculate age)
Country, City
Significant Facts
Number of past relationships
Current relationship details
Additional facts based on test results
info 1
info 2
…
info 50
Onboarding process includes questions that personalize the experience. Information is saved and used for prompt generation.
A database of psychological tests is present:
Specify test availability for Kira, Kira+, or both
Tests can be started with a command, regulated command visibility
Allow interrupting the test with a /stopTest command
Option to discuss test results with a button post-completion
Test discussion messages include test context in chatGPT submissions

Test Characteristics
Test Name, Description, Welcome Image, Number of Questions, Question Details, Post-Question Message
Test Settings: Availability, Command Names, Visibility, Result recording as significant factors, Prompt modification for AI interpretation
Visual Test Generation
Some tests generate imagery based on user description using APIs like DALL-E (implemented).
Content Plan
Messages and tests are sent according to a schedule which checks every 15 minutes.
User filtering for targeted content delivery.
Separate content plans for Kira and Kira+.
Additional Features
Subscription model for Kira+ with payment integration, trial period, and subscription management through customer support.
Shared client information between both versions.



Non-functional Requirements
Development to proceed iteratively, with agreed-upon result expectations for each phase.
No server/bot restart required for new tests or prompt updates; such modifications are to be managed via tools like Google Sheets.
Ensure compatibility with OpenAI APIs using proxies if required.
Log management for both messages and errors, with toggle options.
Message sending limits (token-based) at both general and individual user levels.
The project should include:
README file for start-up instructions
Basic architecture diagram, including database structure
Externalized configuration for easy parameter modification without restarts
Each aspect of the implementation must be consultatively aligned with project requirements and parameters.


=============
Python implementation outline and code structure for developing the Kira and Kira+ Telegram bots using the aiogram library and PostgreSQL as the database. The implementation includes features such as AI-based consultations, psychological tests, and subscription-based features. This structure assumes familiarity with Docker and the use of OpenAI APIs.
Implementation Plan
Tools and Libraries

    aiogram: For Telegram bot development.
    SQLAlchemy or asyncpg: For PostgreSQL database integration.
    Docker: To containerize the bot.
    OpenAI API: For AI consultations.
    Stripe API or PayPal API: For payment integration.

Features

    Kira Free Features:
        AI consultations (static prompts, contextual interaction).
        Basic psychological tests.
        Ads for Kira+.

    Kira+ Features:
        Full psychological tests.
        Advanced AI consultations (e.g., sexology, visual test results).
        Subscription model with payment gateway.

Database Schema

    Users Table: Store user information (age, location, significant facts).
    Tests Table: Store test metadata.
    Subscriptions Table: Manage Kira+ subscriptions.
    Logs Table: Track messages and errors.

Code Implementation
1. Setup Environment

Install required libraries:

pip install aiogram asyncpg openai stripe python-dotenv

Set up the project directory:

project/
├── bot.py
├── db/
│   ├── models.py
│   ├── database.py
├── handlers/
│   ├── start.py
│   ├── tests.py
│   ├── consultations.py
├── config.py
├── Dockerfile
├── docker-compose.yml
└── README.md

2. Core Files
config.py

import os
from dotenv import load_dotenv

load_dotenv()

BOT_TOKEN = os.getenv("BOT_TOKEN")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
POSTGRES_URI = os.getenv("POSTGRES_URI")
STRIPE_API_KEY = os.getenv("STRIPE_API_KEY")
FREE_MESSAGE_LIMIT = int(os.getenv("FREE_MESSAGE_LIMIT", 1000))

database.py

from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import declarative_base

DATABASE_URL = "your_postgres_uri"

engine = create_async_engine(DATABASE_URL, echo=True)
Base = declarative_base()
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

models.py

from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from .database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    telegram_id = Column(String, unique=True, index=True)
    age = Column(Integer)
    city = Column(String)
    significant_facts = Column(String)  # JSON or Text Field
    is_kira_plus = Column(Boolean, default=False)

class Subscription(Base):
    __tablename__ = "subscriptions"
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    start_date = Column(DateTime)
    end_date = Column(DateTime)

class Test(Base):
    __tablename__ = "tests"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String)
    description = Column(String)
    is_kira_plus = Column(Boolean)
    questions = Column(String)  # JSON Field

3. Bot Handlers
start.py

from aiogram import Router, types
from aiogram.dispatcher.filters import Command
from ..db.database import SessionLocal
from ..db.models import User

router = Router()

@router.message(Command("start"))
async def start(message: types.Message):
    async with SessionLocal() as session:
        user = await session.get(User, message.from_user.id)
        if not user:
            new_user = User(telegram_id=message.from_user.id)
            session.add(new_user)
            await session.commit()
    await message.answer("Welcome to Kira! Type /help to see available commands.")

consultations.py

from aiogram import Router, types
import openai
from ..config import OPENAI_API_KEY

openai.api_key = OPENAI_API_KEY
router = Router()

@router.message(Command("consult"))
async def consult(message: types.Message):
    prompt = f"Static prompt with user details: {message.text}"
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=200
    )
    await message.answer(response.choices[0].text.strip())

tests.py

from aiogram import Router, types
from aiogram.dispatcher.filters import Command
from ..db.database import SessionLocal
from ..db.models import Test

router = Router()

@router.message(Command("start_test"))
async def start_test(message: types.Message):
    test_name = message.text.split(" ", 1)[1]
    async with SessionLocal() as session:
        test = await session.query(Test).filter_by(name=test_name).first()
        if not test:
            await message.answer("Test not found!")
        else:
            await message.answer(f"Starting test: {test.description}")

4. Dockerfile

FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "bot.py"]

Key Functionalities

    AI Integration:
        Use OpenAI API to generate context-based responses and test analysis.

    Subscriptions:
        Integrate Stripe for managing Kira+ payments with a trial period.

    Tests:
        Use commands to start tests.
        Store results in the database and feed them into AI prompts for personalized responses.

    Database:
        PostgreSQL stores user info, test metadata, and logs.

Next Steps

    Deploy the bot using Docker and test locally.
    Add logging for debugging and performance monitoring.
    Build the payment system and ensure secure subscription management.

Let me know if you need help with specific parts! 🚀
