
export const handler = async (event) => {
  // TODO implement
  const crypto = require('crypto');
  const fs = require('fs');
   
  // console.log("Encrypted Data:", encryptedData.toString('base64'));
   
  // Load the private key from the PEM file
  const privateKey = fs.readFileSync('private.pem', 'utf8');
   
  // The encrypted data (base64-encoded from the previous step)
  const encryptedData1 = 'AYADeHNjRd9QMpGuKYPswzXxpJMAXwABABVhd3MtY3J5cHRvLXB1YmxpYy1rZXkAREF1TnpFTkNaamVnSldaU041MXBlSTFYOGZQQldzRTNPOXpLOE5TVkpZOC9JZ29uVFhTRWdRazRYZTRxdmU3MEo3Zz09AAEADUFtYXpvbkNvbm5lY3QAJDRlMGU4YmQ0LWExN2UtNGQxNi04Mzg5LWE2MDVlNzRhM2JmOAEAIGbxieE/Lc/2qxoDQJTo/gp++kv1vZyIFFdmPd394h5PxV2r2hfDSMmUlG3UkN0ZWsByzQzJbK1Grhw/EaIbVE7VxQnrjKxKPOV26YenevxF7PrDASl9Wi91JCgkQ0T3Qqxb6fXqDhE8sVP1wuDAlzZlrFEaOzcUrUIs42T7s6KBEVrFs7Kcgj1RZ9jUt5dWgtnABBFTSgOnBIGjdGxBxvKMpJDH6A5RsEv1oET96+ofRhJDjg8VhXrPgy10JDKdn5d0Dxp+8/o5ztW9Qkyy6ikUJQtmO9p2sWs3O4+mVGFuW4AcdAejC47Gqgtnjucxx2HU518Tux3u5ksiXhYVAQIAAAAADAAAEAACiKeekz67p7AeDbhPyqBFTxsWj9k9nm1gRGz+/////wAAAAG0qEujMRpKY6Q5Kl8AAAAJ3r5CC6A9wBCY2Brlos2XKrAVt/loHz287wBnMGUCME00SNpYdI5eDiszMM58iZXbA0GCLqrodHAFKDT5UzBN5RnjxwsxC5L0s5IhgCdQjQIxAN0qy3EJnaLNFm73h5GdqD4k1U455feMITvAvo1e3inZXBhV99VfK+sgbSl8T9qT0A==';
   
  // Convert the base64-encoded encrypted data to a Buffer
  const encryptedBuffer = Buffer.from(encryptedData1, 'base64');
   
  // Decrypt the data using the private key
  const decryptedData = crypto.privateDecrypt(
      {
          key: privateKey,
          padding: crypto.constants.RSA_PKCS1_OAEP_PADDING,
          oaepHash: "sha256",
      },
      encryptedBuffer
  );
   
  console.log("Decrypted Data:", decryptedData.toString());
  const response = {
    statusCode: 200,
    body: JSON.stringify('Hello from Lambda!'),
  };
  return response;
};
