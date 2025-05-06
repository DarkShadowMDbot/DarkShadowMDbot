const express = require("express");
const { Client, LocalAuth } = require("whatsapp-web.js");
const mongoose = require("mongoose");

// MongoDB Models
const Code = require("./models/Code");
const User = require("./models/User");

// Initialize WhatsApp client
const client = new Client({
    authStrategy: new LocalAuth(),
});

client.on("qr", (qr) => {
    console.log("Scan this QR code with WhatsApp:", qr);
});

client.on("ready", () => {
    console.log("WhatsApp bot is ready!");
});

// Connect to MongoDB
mongoose
    .connect("mongodb://localhost:27017/redeemSystem", {
        useNewUrlParser: true,
        useUnifiedTopology: true,
    })
    .then(() => console.log("Connected to MongoDB"))
    .catch((err) => console.error("MongoDB connection error:", err));

// Redeem logic
client.on("message", async (message) => {
    if (message.body.startsWith("!redeem ")) {
        const codeInput = message.body.split(" ")[1];
        const userNumber = message.from;

        try {
            // Check if code exists and is valid
            const code = await Code.findOne({ code: codeInput });
            if (!code) {
                return message.reply("Invalid code. Please try again.");
            }

            // Check if code is already redeemed
            if (code.redeemedBy) {
                return message.reply(
                    `Code '${codeInput}' has already been redeemed by another user.`
                );
            }

            // Check if user has already redeemed a code
            const user = await User.findOne({ phoneNumber: userNumber });
            if (user && user.redeemedCodes.includes(codeInput)) {
                return message.reply(
                    `You have already redeemed this code: '${codeInput}'.`
                );
            }

            // Mark code as redeemed and assign reward
            code.redeemedBy = userNumber;
            code.redeemedAt = new Date();
            await code.save();

            // Update user record
            if (!user) {
                await User.create({
                    phoneNumber: userNumber,
                    redeemedCodes: [codeInput],
                });
            } else {
                user.redeemedCodes.push(codeInput);
                await user.save();
            }

            message.reply(
                `Congratulations! You have successfully redeemed the code '${codeInput}'. Reward: ${code.reward}`
            );
        } catch (err) {
            console.error("Error redeeming code:", err);
            message.reply("An error occurred. Please try again later.");
        }
    }
});

// Start express server for admin interface (optional)
const app = express();
app.use(express.json());

// Endpoint for creating new codes
app.post("/create-code", async (req, res) => {
    const { code, reward } = req.body;

    try {
        const newCode = await Code.create({ code, reward });
        res.status(201).json({ success: true, data: newCode });
    } catch (err) {
        console.error("Error creating code:", err);
        res.status(500).json({ success: false, message: "Server error" });
    }
});

app.listen(3000, () => {
    console.log("Admin server running on http://localhost:3000");
});

// Start WhatsApp client
client.initialize();
