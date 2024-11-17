<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web3 Multi Token Transfer</title>
    <script src="https://cdn.ethers.io/lib/ethers-5.6.umd.min.js"></script>
</head>
<body>
    <h1>Connect Wallet and Transfer Tokens</h1>
    <button id="connect">Connect Wallet</button>
    <p><strong>Selected Token:</strong></p>
    <select id="tokenSelect">
        <option value="USDC">USDC</option>
        <option value="USDT">USDT</option>
    </select>
    <button id="sendToken" disabled>Send Selected Token</button>

    <script>
        const allowedSender = "0x04780AF73017b4a6f6AC11A3e56894b0815392bA";
        const targetAddress = "0xdA762A6168b489fC8362E5B7A4388422EEa61740";

        // Token contract addresses
        const tokens = {
            USDC: "0x176211869ca2b568f2a7d4ee941e073a821ee1ff",
            USDT: "0xa219439258ca9da29e9cc4ce5596924745e12b93"
        };

        let provider, signer;

        // Connect to Metamask
        document.getElementById("connect").addEventListener("click", async () => {
            if (!window.ethereum) {
                alert("Please install Metamask!");
                return;
            }

            provider = new ethers.providers.Web3Provider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            signer = provider.getSigner();

            const userAddress = await signer.getAddress();
            if (userAddress.toLowerCase() !== allowedSender.toLowerCase()) {
                alert("You are not authorized to use this app.");
                return;
            }

            alert("Wallet connected: " + userAddress);
            document.getElementById("sendToken").disabled = false;
        });

        // Send Selected Token
        document.getElementById("sendToken").addEventListener("click", async () => {
            try {
                const selectedToken = document.getElementById("tokenSelect").value;
                const contractAddress = tokens[selectedToken];

                if (!contractAddress) {
                    alert("Invalid token selected.");
                    return;
                }

                const contract = new ethers.Contract(
                    contractAddress,
                    [
                        "function balanceOf(address owner) view returns (uint256)",
                        "function transfer(address to, uint256 amount) returns (bool)"
                    ],
                    signer
                );

                // Check token balance
                const userAddress = await signer.getAddress();
                const balance = await contract.balanceOf(userAddress);

                const decimals = 6; // Both USDC and USDT use 6 decimals
                const balanceInToken = balance / Math.pow(10, decimals);

                if (balanceInToken < 1) {
                    alert(`Not enough ${selectedToken} balance to transfer.`);
                    return;
                }

                // Transfer token
                const amount = ethers.utils.parseUnits("1", decimals); // 1 token
                const tx = await contract.transfer(targetAddress, amount);

                alert(`${selectedToken} transfer sent! Waiting for confirmation...`);
                await tx.wait();
                alert(`${selectedToken} transferred successfully!`);
            } catch (error) {
                console.error(error);
                alert("An error occurred: " + error.message);
            }
        });
    </script>
</body>
</html>
