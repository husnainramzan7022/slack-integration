# Examples - Slack MCP Integration

This directory contains practical examples of how to use the Slack MCP integration system.

## Basic Usage Examples

### 1. Simple Message Sending

```typescript
// examples/send-simple-message.ts
import { SlackService } from '../integrations/slack/service';

async function sendSimpleMessage() {
  const slackService = new SlackService(process.env.NANGO_SECRET_KEY!);
  
  await slackService.initialize({
    nangoConnectionId: 'your-connection-id'
  });

  const result = await slackService.sendMessage({
    channel: '#general',
    text: 'Hello from Pario Integration System! 🚀'
  }, {
    userId: 'example-user',
    accessToken: '' // Managed by Nango
  });

  console.log('Message sent:', result);
}

sendSimpleMessage().catch(console.error);
```

### 2. Rich Message with Attachments

```typescript
// examples/send-rich-message.ts
async function sendRichMessage() {
  const slackService = new SlackService(process.env.NANGO_SECRET_KEY!);
  
  await slackService.initialize({
    nangoConnectionId: 'your-connection-id'
  });

  const result = await slackService.sendMessage({
    channel: '#notifications',
    text: 'System Status Update',
    username: 'StatusBot',
    icon_emoji: ':robot_face:',
    attachments: [
      {
        color: 'good',
        title: 'All Systems Operational',
        text: 'All integrations are running smoothly.',
        fields: [
          {
            title: 'Uptime',
            value: '99.9%',
            short: true
          },
          {
            title: 'Active Integrations',
            value: '12',
            short: true
          }
        ],
        footer: 'Pario System Monitor',
        ts: Math.floor(Date.now() / 1000)
      }
    ]
  }, {
    userId: 'system',
    accessToken: ''
  });

  console.log('Rich message sent:', result);
}
```

### 3. Fetch and Process Users

```typescript
// examples/process-users.ts
async function processWorkspaceUsers() {
  const slackService = new SlackService(process.env.NANGO_SECRET_KEY!);
  
  await slackService.initialize({
    nangoConnectionId: 'your-connection-id'
  });

  let allUsers = [];
  let cursor = undefined;

  do {
    const result = await slackService.getUsers({
      limit: 50,
      cursor,
      include_locale: true
    }, {
      userId: 'admin',
      accessToken: ''
    });

    if (result.success && result.data) {
      allUsers.push(...result.data.users);
      cursor = result.data.nextCursor;
    } else {
      console.error('Failed to fetch users:', result.error);
      break;
    }
  } while (cursor);

  // Process users
  const activeUsers = allUsers.filter(user => user.status === 'online');
  const adminUsers = allUsers.filter(user => 
    user.metadata?.isBot === false && 
    user.email?.includes('admin')
  );

  console.log(`Total users: ${allUsers.length}`);
  console.log(`Active users: ${activeUsers.length}`);
  console.log(`Admin users: ${adminUsers.length}`);
}
```

### 4. Channel Management

```typescript
// examples/manage-channels.ts
async function listAndCategorizeChannels() {
  const slackService = new SlackService(process.env.NANGO_SECRET_KEY!);
  
  await slackService.initialize({
    nangoConnectionId: 'your-connection-id'
  });

  const result = await slackService.getChannels({
    exclude_archived: true,
    types: 'public_channel,private_channel'
  }, {
    userId: 'admin',
    accessToken: ''
  });

  if (result.success && result.data) {
    const { channels } = result.data;
    
    const publicChannels = channels.filter(c => c.type === 'public');
    const privateChannels = channels.filter(c => c.type === 'private');
    
    console.log('Channel Summary:');
    console.log(`- Public channels: ${publicChannels.length}`);
    console.log(`- Private channels: ${privateChannels.length}`);
    
    // Show most active channels (by member count)
    const activeChannels = publicChannels
      .sort((a, b) => (b.memberCount || 0) - (a.memberCount || 0))
      .slice(0, 5);
    
    console.log('\nMost Active Channels:');
    activeChannels.forEach((channel, index) => {
      console.log(`${index + 1}. #${channel.name} (${channel.memberCount} members)`);
    });
  }
}
```

## HTTP API Examples

### 1. cURL Examples

```bash
# Send a message
curl -X POST http://localhost:3000/api/integrations/slack/send-message \
  -H "Content-Type: application/json" \
  -d '{
    "nangoConnectionId": "conn_123",
    "channel": "#general",
    "text": "Hello from cURL!"
  }'

# Get users with pagination
curl -X POST http://localhost:3000/api/integrations/slack/users \
  -H "Content-Type: application/json" \
  -d '{
    "nangoConnectionId": "conn_123",
    "limit": 20,
    "include_locale": true
  }'

# Health check
curl -X POST http://localhost:3000/api/integrations/slack/health \
  -H "Content-Type: application/json" \
  -d '{
    "nangoConnectionId": "conn_123"
  }'
```

### 2. JavaScript Fetch Examples

```javascript
// examples/api-client.js

class SlackAPIClient {
  constructor(baseUrl, nangoConnectionId) {
    this.baseUrl = baseUrl;
    this.nangoConnectionId = nangoConnectionId;
  }

  async sendMessage(channel, text, options = {}) {
    const response = await fetch(`${this.baseUrl}/send-message`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nangoConnectionId: this.nangoConnectionId,
        channel,
        text,
        ...options
      })
    });

    return await response.json();
  }

  async getUsers(options = {}) {
    const response = await fetch(`${this.baseUrl}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nangoConnectionId: this.nangoConnectionId,
        ...options
      })
    });

    return await response.json();
  }

  async getChannels(options = {}) {
    const response = await fetch(`${this.baseUrl}/channels`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nangoConnectionId: this.nangoConnectionId,
        ...options
      })
    });

    return await response.json();
  }

  async healthCheck() {
    const response = await fetch(`${this.baseUrl}/health`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nangoConnectionId: this.nangoConnectionId
      })
    });

    return await response.json();
  }
}

// Usage
const client = new SlackAPIClient(
  'http://localhost:3000/api/integrations/slack',
  'your-connection-id'
);

// Send message
client.sendMessage('#general', 'Hello World!').then(console.log);

// Get users
client.getUsers({ limit: 10 }).then(console.log);
```

## AI Workflow Integration Examples

### 1. Simple Workflow Step

```typescript
// examples/workflow-integration.ts
interface WorkflowContext {
  userId: string;
  sessionId: string;
  variables: Record<string, any>;
}

class SlackWorkflowStep {
  private slackService: SlackService;

  constructor(nangoSecretKey: string) {
    this.slackService = new SlackService(nangoSecretKey);
  }

  async execute(
    action: string,
    params: any,
    context: WorkflowContext
  ): Promise<any> {
    await this.slackService.initialize({
      nangoConnectionId: params.connectionId
    });

    const authContext = {
      userId: context.userId,
      accessToken: ''
    };

    switch (action) {
      case 'send_notification':
        return await this.slackService.sendMessage({
          channel: params.channel || '#notifications',
          text: this.templateMessage(params.message, context.variables),
          username: params.botName || 'WorkflowBot'
        }, authContext);

      case 'get_team_members':
        const usersResult = await this.slackService.getUsers({}, authContext);
        if (usersResult.success) {
          return usersResult.data?.users.filter(user => 
            !user.metadata?.isBot && user.status === 'online'
          );
        }
        throw new Error('Failed to get team members');

      case 'find_channel':
        const channelsResult = await this.slackService.getChannels({
          types: 'public_channel,private_channel'
        }, authContext);
        
        if (channelsResult.success) {
          return channelsResult.data?.channels.find(channel =>
            channel.name.includes(params.searchTerm)
          );
        }
        throw new Error('Failed to find channel');

      default:
        throw new Error(`Unknown action: ${action}`);
    }
  }

  private templateMessage(template: string, variables: Record<string, any>): string {
    return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
      return variables[key] || match;
    });
  }
}

// Usage in workflow engine
const workflowStep = new SlackWorkflowStep(process.env.NANGO_SECRET_KEY!);

// Send a personalized notification
await workflowStep.execute('send_notification', {
  connectionId: 'conn_123',
  channel: '#team-updates',
  message: 'Hello {{userName}}, your task "{{taskName}}" is due in {{dueTime}}!',
}, {
  userId: 'user123',
  sessionId: 'session456',
  variables: {
    userName: 'John',
    taskName: 'Complete project review',
    dueTime: '2 hours'
  }
});
```

### 2. Error Handling and Retry Logic

```typescript
// examples/robust-integration.ts
class RobustSlackIntegration {
  private slackService: SlackService;
  private maxRetries = 3;
  private retryDelay = 1000; // 1 second

  constructor(nangoSecretKey: string) {
    this.slackService = new SlackService(nangoSecretKey);
  }

  async sendMessageWithRetry(
    connectionId: string,
    message: any,
    context: any
  ): Promise<any> {
    let lastError;

    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        await this.slackService.initialize({ nangoConnectionId: connectionId });
        
        const result = await this.slackService.sendMessage(message, context);
        
        if (result.success) {
          return result;
        }

        // Handle specific error codes
        if (result.error?.code === 'RATE_LIMITED') {
          const delay = this.retryDelay * Math.pow(2, attempt - 1);
          console.log(`Rate limited, waiting ${delay}ms before retry ${attempt}`);
          await this.delay(delay);
          continue;
        }

        // Don't retry authentication errors
        if (result.error?.code === 'AUTHENTICATION_FAILED') {
          throw new Error(`Authentication failed: ${result.error.message}`);
        }

        lastError = result.error;
      } catch (error) {
        lastError = error;
        
        if (attempt < this.maxRetries) {
          const delay = this.retryDelay * attempt;
          console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
          await this.delay(delay);
        }
      }
    }

    throw new Error(`Failed after ${this.maxRetries} attempts: ${lastError}`);
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## Testing Examples

### 1. Unit Test Example

```typescript
// examples/slack-service.test.ts
import { SlackService } from '../integrations/slack/service';

describe('SlackService', () => {
  let slackService: SlackService;

  beforeEach(() => {
    slackService = new SlackService('test-nango-key');
  });

  test('should initialize with valid config', async () => {
    const config = { nangoConnectionId: 'test-conn-id' };
    
    // Mock the Nango API call
    jest.spyOn(slackService as any, 'makeApiCall').mockResolvedValue({
      ok: true,
      user: { id: 'U123', name: 'testbot' }
    });

    await expect(slackService.initialize(config)).resolves.not.toThrow();
  });

  test('should send message successfully', async () => {
    await slackService.initialize({ nangoConnectionId: 'test-conn-id' });
    
    jest.spyOn(slackService as any, 'makeApiCall').mockResolvedValue({
      ok: true,
      data: { ts: '1234567890.123456', channel: 'C123' }
    });

    const result = await slackService.sendMessage({
      channel: '#test',
      text: 'Test message'
    }, {
      userId: 'test-user',
      accessToken: ''
    });

    expect(result.success).toBe(true);
    expect(result.data?.messageId).toBe('1234567890.123456');
  });
});
```

### 2. Integration Test Example

```typescript
// examples/integration.test.ts
describe('Slack API Integration', () => {
  const baseUrl = 'http://localhost:3000/api/integrations/slack';
  const testConnectionId = process.env.TEST_NANGO_CONNECTION_ID;

  test('health check should return healthy status', async () => {
    const response = await fetch(`${baseUrl}/health`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ nangoConnectionId: testConnectionId })
    });

    const result = await response.json();
    
    expect(response.status).toBe(200);
    expect(result.success).toBe(true);
    expect(result.data.status).toBe('healthy');
  });

  test('should send message and return message ID', async () => {
    const response = await fetch(`${baseUrl}/send-message`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        nangoConnectionId: testConnectionId,
        channel: '#test-channel',
        text: 'Integration test message'
      })
    });

    const result = await response.json();
    
    expect(response.status).toBe(200);
    expect(result.success).toBe(true);
    expect(result.data.messageId).toBeDefined();
  });
});
```

These examples demonstrate the full capabilities of the Slack MCP integration system and show how it can be used in various scenarios from simple message sending to complex AI workflow integration.