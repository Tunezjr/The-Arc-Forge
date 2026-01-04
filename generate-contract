export default async function handler(req, res) {
  // Enable CORS
  res.setHeader('Access-Control-Allow-Credentials', true);
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,OPTIONS,PATCH,DELETE,POST,PUT');
  res.setHeader(
    'Access-Control-Allow-Headers',
    'X-CSRF-Token, X-Requested-With, Accept, Accept-Version, Content-Length, Content-MD5, Content-Type, Date, X-Api-Version'
  );

  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return;
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { type, params } = req.body;

    if (!type || !params) {
      return res.status(400).json({ error: 'Missing required parameters' });
    }

    // Get API key from environment variable
    const apiKey = process.env.ANTHROPIC_API_KEY;
    
    if (!apiKey) {
      return res.status(500).json({ error: 'API key not configured' });
    }

    // Build prompt based on contract type
    let prompt = '';
    
    if (type === 'token') {
      prompt = `Generate a complete, production-ready ERC-20 token smart contract with the following specifications:
- Token Name: ${params.name}
- Symbol: ${params.symbol}
- Total Supply: ${params.supply}
- Use Solidity version ^0.8.20
- Include standard ERC-20 functions (transfer, approve, transferFrom, balanceOf, allowance)
- Add proper events (Transfer, Approval)
- Make it secure and follow best practices
- Include detailed comments

Return ONLY the Solidity code, no explanations before or after. Start with // SPDX-License-Identifier: MIT`;
    } else if (type === 'nft') {
      prompt = `Generate a complete, production-ready ERC-721 NFT smart contract with the following specifications:
- Collection Name: ${params.name}
- Symbol: ${params.symbol}
- Use Solidity version ^0.8.20
- Include standard ERC-721 functions (mint, transfer, approve, etc.)
- Add proper events
- Include a simple mint function
- Make it secure and follow best practices
- Include detailed comments

Return ONLY the Solidity code, no explanations before or after. Start with // SPDX-License-Identifier: MIT`;
    } else if (type === 'voting') {
      prompt = `Generate a complete, production-ready voting/governance smart contract with the following specifications:
- System Name: ${params.name}
- Use Solidity version ^0.8.20
- Allow creating proposals
- Allow voting on proposals
- Track vote counts
- Prevent double voting
- Include owner controls
- Make it secure and follow best practices
- Include detailed comments

Return ONLY the Solidity code, no explanations before or after. Start with // SPDX-License-Identifier: MIT`;
    } else if (type === 'custom') {
      prompt = `Generate a complete, production-ready smart contract based on this description:
${params.description}

Requirements:
- Use Solidity version ^0.8.20
- Follow security best practices
- Include detailed comments explaining the code
- Make it functional and deployable
- Include appropriate access controls

Return ONLY the Solidity code, no explanations before or after. Start with // SPDX-License-Identifier: MIT`;
    }

    // Call Claude API
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4000,
        messages: [
          {
            role: 'user',
            content: prompt
          }
        ]
      })
    });

    if (!response.ok) {
      const error = await response.text();
      console.error('Claude API error:', error);
      return res.status(response.status).json({ error: 'Failed to generate contract' });
    }

const data = await response.json();
    const contractCode = data.content[0].text;

    // Clean up the code (remove markdown code blocks if present)
    let cleanCode = contractCode.trim();
    if (cleanCode.startsWith('```solidity')) {
      cleanCode = cleanCode.replace(/^```solidity\n/, '').replace(/\n```$/, '');
    } else if (cleanCode.startsWith('```')) {
      cleanCode = cleanCode.replace(/^```\n/, '').replace(/\n```$/, '');
    }

    return res.status(200).json({ 
      success: true,
      code: cleanCode 
    });

  } catch (error) {
    console.error('Server error:', error);
    return res.status(500).json({ 
      error: 'Internal server error',
      message: error.message 
    });
  }
}