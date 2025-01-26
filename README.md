

// import fetch from 'node-fetch';
// export const handler = async (event) => {
//     console.log('Entire event:', JSON.stringify(event, null,2));
//   const VERIFY_TOKEN = "token_123";
//   console.log('new',event) 
//   try {
//       // Handle GET request for webhook verification
//       if (event.httpMethod === "GET") {
//           const queryParams = event.queryStringParameters;
          
//           if (
//               queryParams &&
//               queryParams["hub.mode"] === "subscribe" &&
//               queryParams["hub.verify_token"] === VERIFY_TOKEN
//           ) {
//               console.log("Webhook verified successfully");
//               return {
//                   statusCode: 200,
//                   body: queryParams["hub.challenge"],  // Respond with the challenge
//               };
//           } else {
//               console.error("Verification failed");
//               return {
//                   statusCode: 403,
//                   body: "Forbidden",
//               };
//           }
//       }
//       // Handle POST request for receiving messages
//       if (event.httpMethod === "POST") {
//           const body = JSON.parse(event.body);
//           const entries = body.entry;
//           // Check if the entries are valid
//           if (!entries || entries.length === 0) {
//               return { statusCode: 400, body: 'No entry found' };
//           }
          
//           for (const entry of entries) {
//               const messagingEvents = entry.messaging;
//               if (!messagingEvents || messagingEvents.length === 0) {
//                   continue;
//               }
//               // Process each messaging event (received message)
//               for (const messageEvent of messagingEvents) {
//                   const senderId = messageEvent.sender.id;
//                   const messageText = messageEvent.message.text;
//                   // Logging the received message
//                   console.log(`Received message from ${senderId}: ${messageText}`);
//                   // sending a reply back to the Instagram user (via Facebook API)
//                   await sendMessageToInstagram(senderId, `Thank you for your message: ${messageText}`);
//               }
//           }
//           // Return success response
//           return { statusCode: 200, body: 'EVENT_RECEIVED' };
//       }
//       // If the HTTP method is not GET or POST, return a 404 error
//       return { statusCode: 404, body: 'KMHVC' };
//   } catch (error) {
//       console.error("Error processing webhook:", error);
//       return { statusCode: 500, body: 'Internal Server Error' };
//   }
// };
// // Function to send a reply back to the Instagram user (via Facebook API)
// async function sendMessageToInstagram(senderId, messageText) {
//   const accessToken = 'EAAX8zKLRFn8BO4NcgAnFzaHkFQq116F0JY1cMzJSMyeo8IRmglZBbP4TxwEmGqvprQmZCkg8JKSaylI2o066YYUaFxOntbeHChZCO4D3hAPBHQgMZA5YZCzzwUItHWCaG6u0sRF9HmKVsAC31gsFSHNIZB93ZAJBx4DhTjTNW3HygQwma0eeYEa914HLnRHCwEVKgZDZD';  // Replace with your access token
//   const url = `https://graph.facebook.com/v12.0/me/messages?access_token=${accessToken}`;
//   const requestBody = {
//       recipient: { id: senderId },
//       message: { text: messageText }
//   };
//   try {
//       const response = await fetch(url, {
//           method: 'POST',
//           headers: { 'Content-Type': 'application/json' },
//           body: JSON.stringify(requestBody)
//       });
//       const result = await response.json();
//       console.log('Message sent to Instagram:', result);
//   } catch (error) {
//       console.error('Error sending message to Instagram:', error);
//   }
// }

import fetch from 'node-fetch';
// Declare VERIFY_TOKEN and PAGE_ACCESS_TOKEN globally
const VERIFY_TOKEN = "token_123";
const PAGE_ACCESS_TOKEN = "EAAX8zKLRFn8BO4NcgAnFzaHkFQq116F0JY1cMzJSMyeo8IRmglZBbP4TxwEmGqvprQmZCkg8JKSaylI2o066YYUaFxOntbeHChZCO4D3hAPBHQgMZA5YZCzzwUItHWCaG6u0sRF9HmKVsAC31gsFSHNIZB93ZAJBx4DhTjTNW3HygQwma0eeYEa914HLnRHCwEVKgZDZD";
export const handler = async (event) => {
   try {
       // Handle GET Request (Webhook Validation)
       if (event.httpMethod === "GET") {
           const params = event.queryStringParameters;
           if (params["hub.mode"] === "subscribe" && params["hub.verify_token"] === VERIFY_TOKEN) {
               return {
                   statusCode: 200,
                   body: params["hub.challenge"], // Return the challenge value
               };
           }
           return { statusCode: 403, body: "Forbidden - Validation Failed" }; // Invalid token or mode
       }
       // Handle POST Request (Incoming Events)
       if (event.httpMethod === "POST") {
           const body = JSON.parse(event.body);
           if (body.object === "instagram") {
               // Process each entry in the event
               body.entry.forEach((entry) => {
                   const messages = entry.messaging;
                   // Handle each message
                   messages.forEach(async (messageEvent) => {
                       const senderId = messageEvent.sender.id; // User ID
                       const messageText = messageEvent.message?.text || ""; // Received text message
                       console.log(`Received message: "${messageText}" from Instagram user ${senderId}`);
                       // Send a reply to the user
                       await sendReplyToUser(senderId, `You sent: "${messageText}"`);
                   });
               });
               return {
                   statusCode: 200,
                   body: "EVENT_RECEIVED", // Meta expects this exact response
               };
           }
           return { statusCode: 404, body: "Unsupported Event Type" };
       }
       return { statusCode: 404, body: "Unsupported HTTP Method" };
   } catch (error) {
       console.error("Error processing the event:", error);
       return { statusCode: 500, body: "Internal Server Error" };
   }
};
// Function to Send a Reply to the User
async function sendReplyToUser(senderId, message) {
   const url = `https://graph.facebook.com/v16.0/me/messages?access_token=${PAGE_ACCESS_TOKEN}`;
   const body = {
       recipient: { id: senderId },
       message: { text: message },
   };
   try {
       const response = await fetch(url, {
           method: "POST",
           headers: { "Content-Type": "application/json" },
           body: JSON.stringify(body),
       });
       const data = await response.json();
       if (response.ok) {
           console.log("Reply sent successfully:", data);
       } else {
           console.error("Error sending reply:", data);
       }
   } catch (error) {
       console.error("Error sending reply:", error);
   }
}



