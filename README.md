import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.net.UnknownHostException;
import java.util.Scanner;

public class WebCrawler{
	public static void main(String[] args) {

		// Valores "default" para porta e endereço
		int porta = 80;
		String servidor; //"172.17.2.25";

		Socket socket = null;
		BufferedReader input;
		PrintWriter output;
		int ERROR = 1;


		Scanner sc = new Scanner(System.in);    
		String entrada;    

		System.out.println("Digite a URL");    
		entrada = sc.nextLine();    

		servidor = entrada;

		// Entrada pode ser na linha de comando
		if(args.length == 2) {
			// endereco
			servidor = args[0];
			// porta
			try { 
				porta = Integer.parseInt(args[1]);
			}
			catch (Exception e) {
				System.out.println("porta do servidor = 80 (default)");
				porta = 80;
			}
		} 
		// conecta ao servidor
		try {
			System.out.println("Conectando ao servidor " +
					servidor +
					":" + porta);

			socket = new Socket(servidor, porta);
			System.out.println("Conectado ao servidor " +
					socket.getInetAddress() +
					":" + socket.getPort());
		}
		catch (UnknownHostException e) {
			System.out.println(e);
			System.exit(ERROR);
		}
		catch (IOException e) {
			System.out.println(e);
			System.exit(ERROR);
		}

		try {
			//input = new BufferedReader(new InputStreamReader(System.in)); 
			input = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			output = new PrintWriter(socket.getOutputStream(),true);

			output.println("GET /index.html HTTP/1.1");
			output.println("Host: "+servidor);
			output.println("");

			int qtdGif=0;
			int qtdJpg=0;
			int qtdHttp=0;
			int nivel=1;

			String str = "";
			String strAux = "";

			do{
				//System.out.println(str);

				//Busca GIF
				strAux = str;
				while (strAux.contains(".gif")){
					qtdGif++;
					strAux = strAux.replace(".gif","");
				}

				//Busca JPG
				strAux=str;
				while (strAux.contains(".jpg")){
					qtdJpg++;
					strAux = strAux.replace(".jpg","");
				}

				//Busca http://
				strAux=str;
				while (strAux.contains("href=")){
					int posInicial = strAux.indexOf("href=");
					String aux = strAux.substring(posInicial+6);
					int posFinal = aux.indexOf('"');
					if (posFinal>0){
						String href = aux.substring(0, posFinal);
						if (!href.contains("#")){
							System.out.println((qtdHttp+1)+"  "+href);
							qtdHttp++;
						}
						strAux=strAux.substring(posFinal, strAux.length());
					} else {
						strAux="";
					}
				}

				str = input.readLine();
			} while ((str!=null)&&(str!=""));

			System.out.println("URL: "+entrada);
			System.out.println("Nível: "+nivel);
			System.out.println("Quantidade de imagens gif: "+qtdGif);
			System.out.println("Quantidade de imagens jpg: "+qtdJpg);
			System.out.println("Quantidade de links: "+qtdHttp);
			/*
			// Le dados do teclado e envia ao servidor
			while(true) {
				mensEnvio = input.readLine();
				// para se a entrada for "."
				if(mensEnvio.equals(".")) break;
				output.println(mensEnvio);
			}
			 */
		}
		catch (IOException e) {
			System.out.println(e);
		}

		// fecha conexao
		try {
			socket.close();
		}
		catch (IOException e) {
			System.out.println(e);
		}
	}    
}