char nextChar = (char)mySerialPort.readChar();
byte nextByte = (byte)mySerialPort.readByte();
It is worth noting that both ReadLine(), ReadExisting() return a string based off of decoded bytes from the input buffer.  It means that for example if we received the bytes 0x48, 0x69, and 0x0A those would be decoded based off of the ASCII encoding to ‘H’ , ‘I’ , and ‘\n’
If you wanted to read the actual numeric value you should use readByte() or readChar() since they return integer values which are not decoded. A better way to be continuously reading is to check if there is data to be read in the input buffer using the SerialPort.BytesToRead property. 
The same concept is applied to Write() and WriteLine(). The call WriteLine(“Hi”) writes 0x41, 0x61, 0x0A (0x0A is the added ‘\n’ since we used WriteLine()). If we want to recognize them as characters on the hardware side you must have your own decoding logic present there.

#ReadExist
class SerialPortProgram 
 { 
  // Create the serial port with basic settings 
    private SerialPort port = new   SerialPort("COM1",
      9600, Parity.None, 8, StopBits.One); 
    [STAThread] 
    static void Main(string[] args) 
    { 
      // Instatiate this 
      class new SerialPortProgram(); 
    } 

    private SerialPortProgram() 
    { 
        Console.WriteLine("Incoming Data:");
         // Attach a method to be called when there
         // is data waiting in the port's buffer 
        port.DataReceived += new SerialDataReceivedEventHandler(port_DataReceived); 
        // Begin communications 
        port.Open(); 
        // Enter an application loop to keep this thread alive 
        Application.Run(); 
     } 

    private void port_DataReceived(object sender, SerialDataReceivedEventArgs e) 
    { 
       // Show all the incoming data in the port's buffer
       Console.WriteLine(port.ReadExisting()); 
    } 
}
#BytesToRead

public class MySerialReader : IDisposable
{
   private SerialPort serialPort;
   private Queue<byte> recievedData = new Queue<byte>();

   public MySerialReader()
   {
      serialPort = new SerialPort();
      serialPort.Open();
      serialPort.DataReceived += serialPort_DataReceived;
   }

   void serialPort_DataReceived(object s, SerialDataReceivedEventArgs e)
   {
      byte[] data = new byte[serialPort.BytesToRead];
      serialPort.Read(data, 0, data.Length);
      data.ToList().ForEach(b => recievedData.Enqueue(b));
      processData();
    }

  void processData()
  {
    // Determine if we have a "packet" in the queue
    if (recievedData.Count > 50)
    {
        var packet = Enumerable.Range(0, 50).Select(i => recievedData.Dequeue());
    }
  }

  public void Dispose()
  {
        if (serialPort != null)
        {
            serialPort.Dispose();
        }
  }
