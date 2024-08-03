# test
reg fb
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
using System;
using System.IO;
using System.Threading;
using System.Windows.Forms;
using Newtonsoft.Json;

namespace selenium2
{
    public partial class Form1 : Form
    {
        string ProfileFolderPath = "Profile";
        string ChromeDriverPath = @"C:\Users\ADMIN\source\repos\selenium2\bin\Debug";
        ChromeDriver driver;

        private static Random random = new Random();

        private FilePathData filePathData = new FilePathData();

        public Form1()
        {
            InitializeComponent();
            LoadFilePathsFromJson();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            if (driver != null)
            {
                try
                {
                    driver.Close();
                    driver.Quit();
                }
                catch (Exception)
                {
                }
            }

            ChromeOptions options = new ChromeOptions();
            options.AddArguments("--no-sandbox");
            options.AddArguments("--disable-dev-shm-usage");
            options.AddArguments("--remote-debugging-port=9222");

            if (!Directory.Exists(ProfileFolderPath))
            {
                Directory.CreateDirectory(ProfileFolderPath);
            }

            if (Directory.Exists(ProfileFolderPath))
            {
                options.AddArguments("user-data-dir=" + Path.Combine(ProfileFolderPath, "15"));
            }

            try
            {
                driver = new ChromeDriver(ChromeDriverPath, options);
                driver.Navigate().GoToUrl("https://mbasic.facebook.com/reg/?_rdr");
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error: " + ex.Message);
            }

            try
            {
                string ten = GetRandomLineFromFile(txtTen.Text);
                IWebElement tenElement = driver.FindElement(By.CssSelector("#firstname > div > input"));
                tenElement.SendKeys(ten);
                PauseForSeconds(5);

                string ho = GetRandomLineFromFile(txtHo.Text);
                IWebElement hoElement = driver.FindElement(By.CssSelector("#firstname > div > div.bc > input"));
                hoElement.SendKeys(ho);
                PauseForSeconds(5);

                string phone = GetRandomLineFromFile(txtPhone.Text);
                IWebElement phoneInput = driver.FindElement(By.CssSelector("#contactpoint_step_input"));
                phoneInput.SendKeys(phone);
                PauseForSeconds(5);

                string pass = GetRandomLineFromFile(txtPass.Text);
                IWebElement passElement = driver.FindElement(By.CssSelector("#password_step_input"));
                passElement.SendKeys(pass);
                PauseForSeconds(5);

                Random random = new Random();

                int day = random.Next(1, 29);
                int month = random.Next(1, 13);
                int year = random.Next(1970, 2005);

                IWebElement ngay = driver.FindElement(By.CssSelector("#day"));
                SelectElement daySelector = new SelectElement(ngay);
                daySelector.SelectByValue(day.ToString());

                IWebElement thang = driver.FindElement(By.CssSelector("#month"));
                SelectElement monthSelector = new SelectElement(thang);
                monthSelector.SelectByValue(month.ToString());

                IWebElement nam = driver.FindElement(By.CssSelector("#year"));
                SelectElement yearSelector = new SelectElement(nam);
                yearSelector.SelectByValue(year.ToString());

                PauseForSeconds(5);

                int randomSex = random.Next(1, 4);

                string sexValue = randomSex.ToString();

                IWebElement sexElement = driver.FindElement(By.Name($"sex"));
                sexElement.Click();

                IWebElement sexOption = driver.FindElement(By.CssSelector($"input[name='sex'][value='{sexValue}']"));
                sexOption.Click();

                PauseForSeconds(5);

                IWebElement registerButton = driver.FindElement(By.CssSelector("#signup_button"));
                registerButton.Click();
            }
            catch (NoSuchElementException ex)
            {
                MessageBox.Show("Không tìm thấy phần tử web: " + ex.Message, "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
            catch (Exception ex)
            {
                MessageBox.Show("Đã xảy ra lỗi: " + ex.Message, "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void btnHo_Click(object sender, EventArgs e)
        {
            string hoFilePath = GetFilePath();
            if (hoFilePath != null)
            {
                filePathData.Ho = hoFilePath;
                txtHo.Text = hoFilePath;
                SaveFilePathsToJson();
            }
        }

        private void btnTen_Click(object sender, EventArgs e)
        {
            string tenFilePath = GetFilePath();
            if (tenFilePath != null)
            {
                filePathData.Ten = tenFilePath;
                txtTen.Text = tenFilePath;
                SaveFilePathsToJson();
            }
        }

            private void btnPhone_Click(object sender, EventArgs e)
        {
            string phoneFilePath = GetFilePath();
            if (phoneFilePath != null)
            {
                filePathData.Phone = phoneFilePath;
                txtPhone.Text = phoneFilePath;
                SaveFilePathsToJson();
            }
        }

        private void btnPass_Click(object sender, EventArgs e)
        {
            string passFilePath = GetFilePath();
            if (passFilePath != null)
            {
                filePathData.Pass = passFilePath;
                txtPass.Text = passFilePath;
                SaveFilePathsToJson();
            }
        }

        private static string GetRandomLineFromFile(string filePath)
        {
            var lines = File.ReadAllLines(filePath);
            if (lines.Length == 0)
            {
                throw new Exception($"File {filePath} không chứa dòng nào.");
            }
            int index = random.Next(lines.Length);
            return lines[index];
        }

        private static void PauseForSeconds(int seconds)
        {
            Thread.Sleep(seconds * 500);
        }

        private string GetFilePath() 
        {
            OpenFileDialog openFileDialog = new OpenFileDialog();
            openFileDialog.Title = "Chọn tệp";
            openFileDialog.Filter = "Text files (*.txt)|*.txt|All files (*.*)|*.*";

            if (openFileDialog.ShowDialog() == DialogResult.OK)
            {
                return openFileDialog.FileName;
            }

            return null;
        }

        private void SaveFilePathsToJson()
        {
            try
            {
                string json = JsonConvert.SerializeObject(filePathData, Formatting.Indented);
                File.WriteAllText("filePaths.json", json);
            }
            catch (Exception ex)
            {
                MessageBox.Show("Lỗi khi lưu tệp JSON: " + ex.Message, "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
        private void LoadFilePathsFromJson()
        {
            try
            {
                if (File.Exists("filePaths.json"))
                {
                    string json = File.ReadAllText("filePaths.json");
                    filePathData = JsonConvert.DeserializeObject<FilePathData>(json);

                    txtHo.Text = filePathData.Ho;
                    txtTen.Text = filePathData.Ten;
                    txtPhone.Text = filePathData.Phone;
                    txtPass.Text = filePathData.Pass;
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Lỗi khi tải tệp JSON: " + ex.Message, "Lỗi", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        public class FilePathData
        {
            public string Ho { get; set; }
            public string Ten { get; set; }
            public string Phone { get; set; }
            public string Pass { get; set; }
        }
    }
}
