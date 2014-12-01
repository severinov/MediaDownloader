using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
// --- 
using System.Web;
using System.IO;
using System.Media;
using System.Net;
using System.Security;
using WMPLib;


namespace MediaDownloader
{
    public partial class Form1 : Form
    {
        WMPLib.WindowsMediaPlayer player;

        public Form1()
        {
            InitializeComponent();
            initAppVk();
        }
        private void initAppVk()
        {
            string vkUrl = "https://oauth.vk.com/authorize?client_id=4658286&scope=audio&"+
                           "redirect_uri=http://oauth.vk.com/blank.html&display=mobile&response_type=token&revoke=0";
            
            Uri requestUri = new Uri(vkUrl);
            Uri callbackUri = new Uri("http://oauth.vk.com/blank.html");

            webBrowser1.Url = requestUri;

            player = new WMPLib.WindowsMediaPlayer();

        }

        private void webBrowser1_DocumentCompleted(object sender, WebBrowserDocumentCompletedEventArgs e)
        {
            try
            {
                char[] separators = { '=', '&' };
                string responseString = e.Url.ToString();
                string[] responseContent = responseString.Split(separators);
                string accessToken = responseContent[1];
                int userID = Int32.Parse(responseContent[5]);
                if (userID > 0)
                {
                    webBrowser1.Visible = false;
                    string sURL = "https://api.vk.com/method/audio.get.xml?owner_id="+userID+"&count=100&access_token="+accessToken;
                    WebRequest wrGetUrl = WebRequest.Create(sURL);
                    Stream songsStream  = wrGetUrl.GetResponse().GetResponseStream();
                    StreamReader songsStreamReader = new StreamReader(songsStream);
                    string allText = songsStreamReader.ReadToEnd();

                    if (allText.Length > 0)
                    {
                        File.WriteAllText(@"c:\users\nikita\documents\visual studio 2013\Projects\MediaDownloader\MediaDownloader\songs.xml", allText);
                        DataSet dataSet = new DataSet();
                        dataSet.ReadXml(@"c:\users\nikita\documents\visual studio 2013\Projects\MediaDownloader\MediaDownloader\songs.xml");
                        dataGridView1.DataSource = dataSet.Tables[1];
                        dataGridView1.Columns[0].Visible = false;
                        dataGridView1.Columns[1].Visible = false;
                        dataGridView1.Columns[4].Visible = false;
                        dataGridView1.Columns[6].Visible = false;
                        dataGridView1.Columns[7].Visible = false;

                        DataGridViewButtonColumn downloadBtnColumn = new DataGridViewButtonColumn();
                        downloadBtnColumn.Name = "Action";
                        downloadBtnColumn.UseColumnTextForButtonValue = true;
                        downloadBtnColumn.Text = "Download & Play";

                        if (dataGridView1.Columns["Action"] == null)
                            dataGridView1.Columns.Add(downloadBtnColumn);

                        dataGridView1.Visible = true;
                        progressBar1.Visible  = true;

                        dataGridView1.CellClick += new DataGridViewCellEventHandler(dataGridView1_CellClick);


                        
                    }

                }
            }
            catch
            {

            }
        }

        void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
        {
            if (e.ColumnIndex == 0)
            {
                string absolutePathToAudio = dataGridView1.Rows[e.RowIndex].Cells[6].Value.ToString();
                
                WebClient webClient = new WebClient();
               
                webClient.DownloadFileCompleted   += new AsyncCompletedEventHandler(fileCompleted);
                webClient.DownloadProgressChanged += new DownloadProgressChangedEventHandler(fileProgressChanged);

                string[] parts = absolutePathToAudio.Split('?');
                Uri audioURL = new Uri(parts[0]);

                string nameOfAudio = dataGridView1.Rows[e.RowIndex].Cells[3].Value.ToString() + "-" +
                                     dataGridView1.Rows[e.RowIndex].Cells[4].Value.ToString()+".mp3";

                char[] invalidSymbols = {',','|','"','*','>','<','\\','/','?'};
                for (int i = 0; i < invalidSymbols.Length; i++)
                   nameOfAudio = nameOfAudio.Replace(invalidSymbols[i], ' ');

                webClient.DownloadFileAsync(audioURL, @"C:\Songs\"+nameOfAudio);

                player.controls.stop();
                player.close();
                player.URL = absolutePathToAudio;
                player.controls.play();
                
            }
        }

        void fileProgressChanged(object sender, DownloadProgressChangedEventArgs e)
        {
            progressBar1.Value = e.ProgressPercentage;
        }

        void fileCompleted(object sender, AsyncCompletedEventArgs e)
        {
            player.controls.stop();
            progressBar1.Value = 0;
        }
    }
}
