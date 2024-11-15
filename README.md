import https from 'https';

export const handler = async () => {
    try {
        
        const data = await callApi();
        const processedData = processData(data);

        // Prepare response for Amazon Connect with contact attributes
        const connectResponse = {
            ACTIONS: [
                processedData
            ]
        };

       
        return connectResponse;
    } catch (error) {
        console.error('Error:', error);
        // Return an error response in the expected format
        return {
            ACTIONS: [
                { error: 'Internal server error' }
            ]
        };
    }
};


function callApi() {
    return new Promise((resolve, reject) => {
        https.get('https://jsonplaceholder.typicode.com/users', (res) => {
            let data = '';

            // Collect data chunks
            res.on('data', (chunk) => (data += chunk));

            // Parse JSON when response ends
            res.on('end', () => {
                try {
                    resolve(JSON.parse(data)); // Resolve with parsed JSON data
                } catch (parseError) {
                    reject(`Parsing error: ${parseError}`);
                }
            });
        }).on('error', reject);
    });
}


function processData(data) {
    // Example: We extract the first user's name and email as contact attributes
    const user = data[0]; // Using the first user in the list as an example

    // Create contact attributes to be used in Amazon Connect
    return {
        UserName: user.name,
        UserEmail: user.email,
        UserId: user.id.toString()
    };
}
