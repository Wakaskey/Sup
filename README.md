```js
const { default: makeWASocket, useSingleFileAuthState, DisconnectReason, MessageType } = require('@adiwajshing/baileys');
const { state, saveState } = useSingleFileAuthState('./auth_info.json');
const qrcode = require('qrcode-terminal');

async function startBot() {
  const sock = makeWASocket({
    auth: state,
  });

  sock.ev.on('creds.update', saveState);

  sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect, qr } = update;
    if (qr) qrcode.generate(qr, { small: true });
    if (connection === 'close') {
      if ((lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut) startBot();
      else console.log('Logged out');
    }
    console.log('Connection update:', connection);
  });

  sock.ev.on('messages.upsert', async (m) => {
    const msg = m.messages[0];
    if (!msg.message || msg.key.fromMe) return;

    const sender = msg.key.remoteJid;
    const isGroup = sender.endsWith('@g.us');
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text;

    if (!text) return;

    if (text === 'ping') {
      await sock.sendMessage(sender, { text: 'pong' });
    }

    if (text === '.tagall' && isGroup) {
