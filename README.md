# Animales
Codificacion del Prototy
///Como se implementa el primer codifo de la aplicacion///

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Drawing.Imaging;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using AForge.Math.Metrics;
using AForge.Math.Geometry;
using AForge.Math.Random; 
using AForge.Imaging;
using AForge.Imaging.Filters;
using AForge.Video;
using AForge.Video.DirectShow;
using AForge.Vision.Motion;
using AForge;

namespace Capturas
{
    public partial class Form1 : Form
    {
        //variable para tipo de dispositivo
        private FilterInfoCollection videoDevices;
        //variable para fuente de video
        private VideoCaptureDevice videoSource;
        //Variable para la deteccion
        MotionDetector  Destructor;
        private float NdeDeteccion;
        private Color color;
       // private float hue;
        

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            //Inicializar variable de Destructor
            Destructor = new MotionDetector(new TwoFramesDifferenceDetector(), new MotionBorderHighlighting());
            NdeDeteccion = 0;
            
            videoDevices = new FilterInfoCollection(FilterCategory.VideoInputDevice);

            foreach (FilterInfo device in videoDevices)
            {

                comboBox1.Items.Add(device.Name);
            }
             //comboBox1.SelectedIndex = 0;

            videoSource = new VideoCaptureDevice();

        }

        private void button1_Click(object sender, EventArgs e)
        {
            if (videoSource.IsRunning)
            {
               videoSource.Stop();
               pictureBox1.Image = null;
               pictureBox1.Invalidate(); 
             }
            else
            {
                videoSource = new VideoCaptureDevice(videoDevices[comboBox1.SelectedIndex].MonikerString);
                // set NewFrame event hadler 
                videoSource.NewFrame += new NewFrameEventHandler (videoSource_NewFrame);
                videoSource.Start(); 
            }
         
        }

        void videoSource_NewFrame(object sender, NewFrameEventArgs eventArgs)
        {
            Bitmap image = (Bitmap)eventArgs.Frame.Clone();
            Bitmap x = image.Clone() as Bitmap;
            NdeDeteccion = Destructor.ProcessFrame(image);
            pictureBox1.Image = image;
            EuclideanColorFiltering filter = new EuclideanColorFiltering();
            filter.CenterColor = new AForge.Imaging.RGB(color.R, color.G, color.B);
            filter.Radius = 90;
            filter.ApplyInPlace(x);
            
            // create filter
            ColorFiltering colorFilter = new ColorFiltering();
            // configure the filter
            colorFilter.Red = new  IntRange(0, 100);
            colorFilter.Green = new IntRange(0, 200);
            colorFilter.Blue = new IntRange(150, 255);
            // apply the filter
            Bitmap objectImage = colorFilter.Apply(image);
           
           // Color col = x.GetPixel(320, 240);
            
            //if (col.R > col.G && col.R > col.B)
            //{
              //  MessageBox.Show("si");
            //}

            //GrayscaleToRGB  grayscaleFilter = new GrayscaleToRGB();
            //Bitmap grayImage = grayscaleFilter.Apply(x);
            BlobCounter BlobCounter = new BlobCounter();
            BlobCounter.MinWidth = 15;
            BlobCounter.MaxWidth = 15;
            BlobCounter.FilterBlobs = true;
            BlobCounter.ObjectsOrder = ObjectsOrder.Size;
            
            BlobCounter.ProcessImage(x);
            
            Rectangle[] rects = BlobCounter.GetObjectsRectangles();
            foreach (Rectangle recs in rects)
                
                if (rects.Length > 0)
                {
                  Rectangle objectRect = rects[0];
                  Graphics g= Graphics.FromImage(image);
                  Pen pen = new Pen(Color.FromArgb(160, 255, 160), 5);
                  Font fuente = new Font("Arial", 8.0f);
                  {
                      g.DrawRectangle(pen, objectRect);
                  }
                  g.Dispose();  
                }

            pictureBox1.Image = image;

        } 

        private void Form1_FormClosing(object sender, FormClosingEventArgs e)
        {
            if (videoSource.IsRunning)
                videoSource.Stop();
            
        }


        public Rectangle objectRect { get; set; }


        private void button2_Click(object sender, EventArgs e)
        {
            ColorDialog colorD = new ColorDialog();
            colorD.ShowDialog();
            color = (Color)colorD.Color;
            NdeDeteccion = color.GetHue();

        }


    }
}
