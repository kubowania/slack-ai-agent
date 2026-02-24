# Slack AI Agent

An intelligent Slack bot that automatically researches new community members and analyzes their fit for your commercial product using OpenAI and Langchain.

## Features

- **Automatic Member Detection**: Monitors Slack for new member joins
- **AI-Powered Research**: Researches members using multiple sources (GitHub, company websites, etc.)
- **Intelligent Analysis**: Uses GPT-4 to analyze member fit for your product
- **Private Reporting**: Sends formatted analysis reports to a private Slack channel
- **Comprehensive Logging**: Full logging and monitoring for production use
- **Easy Deployment**: Ready-to-deploy on Render.com

## Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│    Slack API     │────▶│  Slack Service   │────▶│ Member Research  │
│  (New Members)   │     │ (Event Handler)  │     │     Service      │
└──────────────────┘     └──────────────────┘     └───────┬──────────┘
                                                          │
┌──────────────────┐     ┌──────────────────┐     ┌───────▼──────────┐
│ Private Channel  │◀────│ Analysis Report  │◀────│ OpenAI/Langchain │
│   (Slack Bot)    │     │    Generator     │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
```

## Prerequisites

- Node.js 18+ 
- OpenAI API key
- Slack workspace with admin access
- Render.com account (for deployment)

## Quick Start

### 1. Clone and Install

```bash
git clone <your-repo-url>
cd slack-ai-agent
npm install
```

### 2. Set Up Slack App

1. Go to [Slack API](https://api.slack.com/apps)
2. Click "Create New App" → "From scratch"
3. Enter app name and select your workspace
4. Navigate to **OAuth & Permissions**:
   - Add these Bot Token Scopes:
     - `channels:read`
     - `chat:write`
     - `users:read`
     - `users:read.email`
   - Install app to workspace
   - Copy the **Bot User OAuth Token** (starts with `xoxb-`)

5. Navigate to **Socket Mode**:
   - Enable Socket Mode
   - Create an App-Level Token with `connections:write` scope
   - Copy the **App-Level Token** (starts with `xapp-`)

6. Navigate to **Event Subscriptions**:
   - Enable Events
   - Subscribe to these bot events:
     - `team_join`
     - `member_joined_channel`

7. Navigate to **Basic Information**:
   - Copy the **Signing Secret**

### 3. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
# Slack Configuration
SLACK_BOT_TOKEN=xoxb-your-bot-token-here
SLACK_APP_TOKEN=xapp-your-app-token-here
SLACK_SIGNING_SECRET=your-signing-secret-here
SLACK_PRIVATE_CHANNEL_ID=C1234567890  # Channel ID where reports are sent

# OpenAI Configuration
OPENAI_API_KEY=sk-your-openai-api-key-here

# Company Information
COMPANY_NAME=Your Company Name
COMPANY_PRODUCT=Your Commercial Product Name
COMPANY_WEBSITE=https://yourcompany.com
COMPANY_DESCRIPTION=Brief description of what your company does
```

### 4. Find Your Private Channel ID

In Slack:
1. Go to your private channel
2. Right-click on the channel name → "Copy Link"
3. The URL will be like: `https://yourworkspace.slack.com/archives/C1234567890`
4. The `C1234567890` part is your channel ID

### 5. Run Locally

```bash
npm run dev
```

The agent will start and display:
```
🚀 Express server running on port 3000
⚡️ Slack bot is running on port 3000  
🎉 Slack AI Agent is fully operational!
```

### 6. Test the Agent

**Method 1: Add a test member to your Slack workspace**

**Method 2: Use the test endpoint (development only):**

```bash
curl -X POST http://localhost:3000/test/analyze-member \
  -H "Content-Type: application/json" \
  -d '{
    "memberInfo": {
      "name": "John Doe",
      "email": "john@techcorp.com",
      "title": "Senior Software Engineer at TechCorp"
    }
  }'
```

## Deployment to Render.com

### 1. Prepare Your Repository

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-github-repo-url>
git push -u origin main
```

### 2. Deploy on Render

1. Go to [Render.com](https://render.com)
2. Click "New" → "Blueprint"
3. Connect your GitHub repository (or use the Blueprint link: `https://dashboard.render.com/blueprint/new?repo=<your-repo-https-url>`)
4. Render will automatically detect the `render.yaml` file
5. Set your environment variables in the Render dashboard:
   - `SLACK_BOT_TOKEN`
   - `SLACK_APP_TOKEN`
   - `SLACK_SIGNING_SECRET`
   - `SLACK_PRIVATE_CHANNEL_ID`
   - `OPENAI_API_KEY`
   - Update company information variables
6. Deploy!

**Why Render Blueprint?** Render detects your `render.yaml` and automatically creates the services and environment variables defined in it. You don’t add the service or env keys by hand because, the Blueprint creates them from the file. You only fill in the secret values (e.g. Slack and OpenAI keys) when prompted. Everything else (build command, start command, health check, plan) is applied from the file, so deployment is reproducible and easy to update by changing the file and pushing.

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `SLACK_BOT_TOKEN` | ✅ | Bot token from Slack app |
| `SLACK_APP_TOKEN` | ✅ | App-level token for Socket Mode |
| `SLACK_SIGNING_SECRET` | ✅ | Signing secret from Slack app |
| `SLACK_PRIVATE_CHANNEL_ID` | ✅ | Channel ID for reports |
| `OPENAI_API_KEY` | ✅ | OpenAI API key |
| `COMPANY_NAME` | ❌ | Your company name |
| `COMPANY_PRODUCT` | ❌ | Your product name |

### Research Sources

The agent researches new members using:

- **Company websites** (from email domains)
- **GitHub profiles** (public repositories and activity)
- **LinkedIn profiles** (placeholder - requires API access)
- **Twitter/X profiles** (placeholder - requires API access)
- **General web search** (placeholder - requires search API)

> **Note**: Some research sources are placeholder implementations. For production use, consider integrating:
> - LinkedIn API (requires approval)
> - Google Custom Search API
> - Clearbit or FullContact APIs
> - ZoomInfo or Apollo.io APIs

## API Endpoints

### Health Check
```
GET /health
```
Returns service health status.

**Why it matters on Render:** The Blueprint sets `healthCheckPath: /health` in `render.yaml`, so Render calls this endpoint after each deploy. When it gets a 2xx (this handler returns 200 with `{ status: 'healthy', timestamp }`), Render marks the deploy live and sends traffic to it; otherwise it can retry or fail the deploy. That enables zero-downtime deploys and faster feedback when the app isn’t ready.

### Status Check
```
GET /status  
```
Returns detailed service status including Slack connection.

### Test Analysis (Development Only)
```
POST /test/analyze-member
Content-Type: application/json

{
  "memberInfo": {
    "name": "John Doe",
    "email": "john@example.com", 
    "title": "Software Engineer"
  }
}
```

## Analysis Output

When a new member joins, the agent sends a report to your private channel:

```
🔍 New Member Analysis: John Doe

Fit Score: 75/100
Email: john@techcorp.com
Title: Senior Software Engineer
Timezone: America/New_York

Key Insights:
• Senior engineer with 5+ years experience
• Works at a mid-size tech company
• Active on GitHub with multiple projects
• Located in target market region

Recommendations:  
• Engage with technical content and demos
• Highlight enterprise features and scalability
• Consider direct outreach for pilot program
• Monitor for questions about integrations

Research Sources:
• Company Website: TechCorp Inc.
• GitHub Profile: john-doe
```

## Monitoring & Logging

The agent includes comprehensive logging:

- **Development**: Console output with colors
- **Production**: File-based logging (`logs/` directory)
- **Structured logs**: JSON format for easy parsing
- **Error tracking**: Automatic error capture and reporting
- **Performance metrics**: Request timing and analysis metrics

## Security Considerations

- ✅ Environment variables for all secrets
- ✅ Input validation and sanitization  
- ✅ Error handling without exposing internals
- ✅ Rate limiting for AI API calls
- ✅ Minimal permission scopes for Slack bot
- ⚠️ Research respects privacy laws (GDPR, CCPA)
- ⚠️ No storage of personal data beyond analysis

## Troubleshooting

### Common Issues

**"Configuration validation failed"**
- Check all required environment variables are set
- Verify Slack tokens are correct format

**"Slack bot not receiving events"**
- Ensure Socket Mode is enabled in Slack app
- Check Event Subscriptions are configured
- Verify bot is added to relevant channels

**"OpenAI API errors"**  
- Verify API key is valid and has credits
- Check for rate limiting (wait and retry)
- Ensure model access (GPT-4 may require approval)

**"No research results"**
- Research failures are handled gracefully
- Check network connectivity for external APIs
- Some research sources are placeholders

### Debugging

Enable debug logging:
```bash
NODE_ENV=development npm start
```

View application logs:
```bash
tail -f logs/combined.log
tail -f logs/error.log  
```

## Development

### Project Structure

```
src/
├── config/          # Configuration management
├── services/        # Core business logic
│   ├── slackService.js       # Slack integration
│   └── memberResearchService.js  # AI research & analysis
└── utils/           # Utilities
    ├── logger.js            # Logging configuration  
    └── researchUtils.js     # Research helpers
```

### Adding New Research Sources

1. Extend `ResearchUtils` class in `src/utils/researchUtils.js`
2. Add new search methods following existing patterns
3. Update `MemberResearchService` to use new sources
4. Test thoroughly and handle API failures gracefully

### Customizing Analysis Prompts

Edit prompts in `src/services/memberResearchService.js`:

- `analysisPrompt`: Main analysis logic
- `researchPrompt`: Research data processing

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, please:
1. Check the troubleshooting section
2. Search existing GitHub issues
3. Create a new issue with detailed information

---

**⚠️ Important**: This agent processes personal information. Ensure compliance with applicable privacy laws and your organization's data policies.

