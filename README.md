float g_Allowances;
float g_Deductions;
unsigned char g_Rand_No[8] = {25};
void Compute_Random_Number(struct AU1_SaPDU *a);
void Read_Emp_Details(struct Emp_data *empdata, int l_n);
void Compute_Allowances(struct Emp_data *empdata, int l_n);
void Compute_Deductions(struct Emp_data *empdata, int l_n);
void Compute_Salary(struct Emp_data *empdata, int l_n);
void Read_Std_Data(struct Std_data *stddata, int l_n);
void Compute_Std_Total(struct Std_data *stddata, int l_n);
void Compute_Pass_Fail(struct Std_data *stddata, int l_n);
void Compute_Average(struct Std_data *stddata, int l_n);
void Compute_Division(struct Std_data *stddata, int l_n);
void print_Std_Data(struct AU1_SaPDU *C, struct Std_data *stddata);
void Display_Emp_Data(struct AU1_SaPDU *b, struct Emp_data *empdata);
enum Enum_Division {
    Distincation,
    First,
    Second,
    Pass,
    NA
};

enum boolean {
    pass,
    Fail
};

struct Data_Obj {
    float Conversion_Factor;
    int Container_Value;
};

struct Std_Performance {
    int Std_Total;
    float Std_Average;
    enum Enum_Division Std_Division;
    enum boolean Std_Pass_Fail;
};

struct Std_Marks {
    int Marks[5];
    struct Std_Performance Std_Score;
};

struct Std_data {
    int Std_No;
    char Std_Name[50];
    struct Std_Marks Std_Marks_Data;
};

struct Emp_Pay {
    float Emp_PT;
    float Emp_SalaryTax;
    float Emp_Salary;
};

struct Emp_Sal {
    int Emp_Basic;
    float Emp_HRA;
    float Emp_DA;
    float Emp_CCA;
    struct Emp_Pay EmpPay;
};

struct Emp_data {
    int EmpNo;
    char EmpName[50];
    struct Emp_Sal EmpBasic;
};

struct Control_Pdu {
    unsigned int Direction_Flag:1;
    unsigned int MTI:12;
    unsigned int Label:6;
    unsigned int reserved:1;
    unsigned int ETY:11;
};


struct AU1_SaPDU {
    union {
        int Control_Pdu;
        struct Control_Pdu control_pdu;
    };
    int ETCS_ID;
    char Random_number[8];
    struct Data_Obj Data;
};

void Compute_Random_Number(struct AU1_SaPDU *a)
{

    int l_i;
    for (l_i = 0; l_i < 8; l_i++) {
      if (l_i == 0) {
         a->Random_number[l_i] = 25;
         } else {
            a->Random_number[l_i] =
            (a->Random_number[l_i - 1] * l_i * 12) % 255;
         }
     }
    printf("Enter the Control_Pdu:");
    scanf("%x",&a->Control_Pdu);
    a->control_pdu.Direction_Flag=(a->Control_Pdu >>0) & 0x1;
    a->control_pdu.MTI=(a->Control_Pdu >>1) & 0xFFF;
    a->control_pdu.Label=(a->Control_Pdu >> 13) & 0x3F;
    a->control_pdu.reserved=(a->Control_Pdu >>1) & 0x1;
    a->control_pdu.ETY=(a->Control_Pdu >>20) & 0x7FF;
    printf("Enter the ETCS_ID:");
    scanf("%d",&a->ETCS_ID);
    if(a->control_pdu.Direction_Flag==1)
    {
        a->Data.Conversion_Factor= a->control_pdu.MTI*2.5;
    }
    if(a->control_pdu.ETY<100)
    {
       a-> Data.Container_Value=(a->control_pdu.MTI*a->control_pdu.ETY/10)+a->control_pdu.Label;
    }
    if((a->control_pdu.ETY>=100)&& (a->control_pdu.ETY<=500))
    {
        a->Data.Container_Value=(a->control_pdu.MTI*a->control_pdu.ETY/100)+a->control_pdu.Label;
    }
    if(a->control_pdu.ETY>500)
    {
        a->Data.Container_Value=(a->control_pdu.MTI*a->control_pdu.ETY/200)+a->control_pdu.Label;
    }
    printf("Direction_Flag=%u\nMTI=%u\n",a->control_pdu.Direction_Flag,a->control_pdu.MTI);
    printf("Label=%u\nETY=%u\n",a->control_pdu.Label,a->control_pdu.ETY);
    printf("Conversion_Factor %f \n", a->Data.Conversion_Factor);
   printf("Container_Value = %d\n", a->Data.Container_Value);
   printf("Random_number array values are : \n");
   for (l_i = 0; l_i < 8; l_i++)
    {
      printf("Random_number[%d] = %d \n", l_i,a->Random_number[l_i]);
   }
}
void Read_Emp_Details(struct Emp_data *empdata, int l_n)
{
    int l_i,l_sal,l_inval=0;
   for(l_i=0;l_i<l_n;l_i++)
   {
       printf("Enter the Employee Name:");
       scanf("%s",&empdata[l_i].EmpName);
       printf("Enter the Employee Number:");
       scanf("%s",&empdata[l_i].EmpNo);
       salary:
       printf("Enter the salary:");
       scanf("%d",&l_sal);
       if(l_sal>2000 && l_sal<200000)
       {
           empdata[l_i].EmpBasic.Emp_Basic=l_sal;
       }
       else
       {
           printf("Invalid salary.Enter valid salary:");
           l_inval++;
           if(l_inval<3)
           {
               goto salary;
           }
           else
           {
               empdata[l_i].EmpBasic.Emp_Basic=0;
           }
       }

   }
}
void Compute_Allowances(struct Emp_data *empdata, int l_n)
{
     int l_i;
    for(l_i=0;l_i<l_n;l_i++)
    {
        if(empdata[l_i].EmpBasic.Emp_Basic<10000)
        {
           g_Allowances=(empdata[l_i].EmpBasic.Emp_Basic*0.3)+(empdata[l_i].EmpBasic.Emp_Basic*0.2)+(empdata[l_i].EmpBasic.Emp_Basic*0.05);
           empdata[l_i].EmpBasic.Emp_HRA=empdata[l_i].EmpBasic.Emp_Basic*0.3;
           empdata[l_i].EmpBasic.Emp_DA=empdata[l_i].EmpBasic.Emp_Basic*0.2;
           empdata[l_i].EmpBasic.Emp_CCA=empdata[l_i].EmpBasic.Emp_Basic*0.05;

        }
       else if(empdata[l_i].EmpBasic.Emp_Basic<40000)
        {
            g_Allowances=(empdata[l_i].EmpBasic.Emp_Basic*0.35)+(empdata[l_i].EmpBasic.Emp_Basic*0.25)+(empdata[l_i].EmpBasic.Emp_Basic*0.15);
           empdata[l_i].EmpBasic.Emp_HRA=empdata[l_i].EmpBasic.Emp_Basic*0.35;
           empdata[l_i].EmpBasic.Emp_DA=empdata[l_i].EmpBasic.Emp_Basic*0.25;
           empdata[l_i].EmpBasic.Emp_CCA=empdata[l_i].EmpBasic.Emp_Basic*0.15;

        }
         else if(empdata[l_i].EmpBasic.Emp_Basic<100000)
        {
            g_Allowances=(empdata[l_i].EmpBasic.Emp_Basic*0.50)+(empdata[l_i].EmpBasic.Emp_Basic*0.35)+(empdata[l_i].EmpBasic.Emp_Basic*0.25);
           empdata[l_i].EmpBasic.Emp_HRA=empdata[l_i].EmpBasic.Emp_Basic*0.50;
           empdata[l_i].EmpBasic.Emp_DA=empdata[l_i].EmpBasic.Emp_Basic*0.35;
           empdata[l_i].EmpBasic.Emp_CCA=empdata[l_i].EmpBasic.Emp_Basic*0.25;

        }
         else if(empdata[l_i].EmpBasic.Emp_Basic<150000)
        {
           g_Allowances=(empdata[l_i].EmpBasic.Emp_Basic*0.52)+(empdata[l_i].EmpBasic.Emp_Basic*0.37)+(empdata[l_i].EmpBasic.Emp_Basic*0.30);
           empdata[l_i].EmpBasic.Emp_HRA=empdata[l_i].EmpBasic.Emp_Basic*0.52;
           empdata[l_i].EmpBasic.Emp_DA=empdata[l_i].EmpBasic.Emp_Basic*0.37;
           empdata[l_i].EmpBasic.Emp_CCA=empdata[l_i].EmpBasic.Emp_Basic*0.30;

        }
         else if(empdata[l_i].EmpBasic.Emp_Basic<200001)
        {
            g_Allowances=(empdata[l_i].EmpBasic.Emp_Basic*0.55)+(empdata[l_i].EmpBasic.Emp_Basic*0.40)+(empdata[l_i].EmpBasic.Emp_Basic*0.32);
           empdata[l_i].EmpBasic.Emp_HRA=empdata[l_i].EmpBasic.Emp_Basic*0.55;
           empdata[l_i].EmpBasic.Emp_DA=empdata[l_i].EmpBasic.Emp_Basic*0.40;
           empdata[l_i].EmpBasic.Emp_CCA=empdata[l_i].EmpBasic.Emp_Basic*0.32;

        }
        else
        {
            g_Allowances=0;
        }
    }
}
void Compute_Deductions(struct Emp_data *empdata, int l_n)
{
   int l_i;
    float l_tol_sal;
    for(l_i=0;l_i<l_n;l_i++)
    {
        l_tol_sal=(empdata[l_i].EmpBasic.Emp_Basic)+(empdata[l_i].EmpBasic.Emp_HRA)+(empdata[l_i].EmpBasic.Emp_DA)+(empdata[l_i].EmpBasic.Emp_CCA);
        if(l_tol_sal<10000)
        {
            empdata[l_i].EmpBasic.EmpPay.Emp_PT=empdata[l_i].EmpBasic.Emp_Basic*0.03;
            empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax=empdata[l_i].EmpBasic.Emp_Basic*0.2;
            g_Deductions=(empdata[l_i].EmpBasic.Emp_Basic*0.03)+(empdata[l_i].EmpBasic.Emp_Basic*0.2);
        }
       else if(l_tol_sal<40000)
        {
            empdata[l_i].EmpBasic.EmpPay.Emp_PT=empdata[l_i].EmpBasic.Emp_Basic*0.035;
            empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax=empdata[l_i].EmpBasic.Emp_Basic*0.25;
            g_Deductions=(empdata[l_i].EmpBasic.Emp_Basic*0.035)+(empdata[l_i].EmpBasic.Emp_Basic*0.25);
        }
        else if(l_tol_sal<100000)
        {
            empdata[l_i].EmpBasic.EmpPay.Emp_PT=empdata[l_i].EmpBasic.Emp_Basic*0.05;
            empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax=empdata[l_i].EmpBasic.Emp_Basic*0.35;
            g_Deductions=(empdata[l_i].EmpBasic.Emp_Basic*0.05)+(empdata[l_i].EmpBasic.Emp_Basic*0.35);
        }
        else if(l_tol_sal<150000)
        {
            empdata[l_i].EmpBasic.EmpPay.Emp_PT=empdata[l_i].EmpBasic.Emp_Basic*0.052;
            empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax=empdata[l_i].EmpBasic.Emp_Basic*0.37;
            g_Deductions=(empdata[l_i].EmpBasic.Emp_Basic*0.052)+(empdata[l_i].EmpBasic.Emp_Basic*0.37);
        }
        else if(l_tol_sal<340000)
        {
            empdata[l_i].EmpBasic.EmpPay.Emp_PT=empdata[l_i].EmpBasic.Emp_Basic*0.055;
            empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax=empdata[l_i].EmpBasic.Emp_Basic*0.40;
            g_Deductions=(empdata[l_i].EmpBasic.Emp_Basic*0.055)+(empdata[l_i].EmpBasic.Emp_Basic*0.40);
        }
        else
        {
            g_Deductions=0;
        }
    }
}
void Compute_Salary(struct Emp_data *empdata, int l_n)
{
     int l_i;
     for(l_i=0;l_i<l_n;l_i++)
     {
        empdata[l_i].EmpBasic.EmpPay.Emp_Salary=(empdata[l_i].EmpBasic.Emp_Basic+g_Allowances)-g_Deductions;
     }
}
void Read_Std_Data(struct Std_data *stddata, int l_n)
{
   int l_i,l_j,l_marks,l_c=0;
    for(l_i;l_i<l_n;l_i++)
    {
        printf("\nEnter student name :  ");
      scanf("%s",&stddata[l_i].Std_Name);
      printf("Enter student roll.num :");
      scanf("%d",&stddata[l_i].Std_No);
        marks:
        for(l_j=0;l_j<5;l_j++) {
         printf("Enter subject %d marks :",l_j+1);
         scanf("%d",&l_marks);
         if(l_marks>=0 &&l_marks<=100)
         {
            stddata[l_i].Std_Marks_Data.Marks[l_j]=l_marks;
			}
			else
			{
				printf("You Entered invalid marks");
            l_c++;
            if(l_c<3)
			   {
				   goto marks;
			   }
			   else
			   {
			   	stddata[l_i].Std_Marks_Data.Marks[0]=1005;
			   	stddata[l_i].Std_Marks_Data.Marks[1]=1005;
			   	stddata[l_i].Std_Marks_Data.Marks[2]=1005;
			   	stddata[l_i].Std_Marks_Data.Marks[3]=1005;
			   	stddata[l_i].Std_Marks_Data.Marks[4]=1005;
			   	break;
				}
			}
      }


    }
}
void Compute_Std_Total(struct Std_data *stddata, int l_n)
{
   int l_i,l_j;
    for(l_i=0;l_i<l_n;l_i++)
    {
        stddata[l_i].Std_Marks_Data.Std_Score.Std_Total=0;
    for(l_j=0;l_j<5;l_j++)
    {
        stddata[l_i].Std_Marks_Data.Std_Score.Std_Total+=stddata[l_i].Std_Marks_Data.Marks[l_j];
    }
  }
}
void Compute_Pass_Fail(struct Std_data *stddata, int l_n)
{
   int l_i,l_j;
    for(l_i=0;l_i<l_n;l_i++)
    {
       if(stddata[l_i].Std_Marks_Data.Marks[l_i]>=35)
       {
           stddata[l_i].Std_Marks_Data.Std_Score.Std_Pass_Fail=pass;
       }
       else
       {
           stddata[l_i].Std_Marks_Data.Std_Score.Std_Pass_Fail=Fail;
       }
    }
}
void Compute_Average(struct Std_data *stddata, int l_n)
{
     int l_i;
    for(l_i=0;l_i<l_n;l_i++)
    {
        stddata[l_i].Std_Marks_Data.Std_Score.Std_Average=stddata[l_i].Std_Marks_Data.Std_Score.Std_Total/5.0;
    }
}
void Compute_Division(struct Std_data *stddata, int l_n)
{
   int l_i,l_grade;
    for(l_i=0;l_i<l_n;l_i++)
    {
        if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Pass_Fail==pass)
        {
            if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Average>80)
            {
                l_grade= 1;
            }
            else if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Average>70)
            {
                l_grade= 2;
            }
            else if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Average>60)
            {
                l_grade= 3;
            }
            else if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Average>40)
            {
                l_grade= 4;
            }

                }
                else{
                    l_grade=5;
                }
        switch(l_grade)
        {
        case 1:
            printf("Rank : Distinction");
            break;
        case 2:
            printf("Rank : First");
            break;
        case 3:
            printf("Rank : Second");
            break;
        case 4:
            printf("Rank : Pass");
            break;
        case 5:
           printf(" Rank:Fail");
           break;
    }
  }
}
void print_Std_Data(struct AU1_SaPDU *c, struct Std_data *stddata)
{
   int l_i,l_n;
  if(c->control_pdu.ETY==500)
  {
      l_n=10;
      Read_Std_Data(stddata,l_n);
      Compute_Std_Total(stddata,l_n);
      Compute_Pass_Fail(stddata,l_n);
      Compute_Average(stddata,l_n);

  }
   else if(c->control_pdu.ETY==400)
  {
    l_n=7;
     Read_Std_Data(stddata,l_n);
      Compute_Std_Total(stddata,l_n);
      Compute_Pass_Fail(stddata,l_n);
      Compute_Average(stddata,l_n);

  }
  else if(c->control_pdu.ETY==300)
  {
      l_n=5;
      Read_Std_Data(stddata,l_n);
      Compute_Std_Total(stddata,l_n);
      Compute_Pass_Fail(stddata,l_n);
      Compute_Average(stddata,l_n);

  }
  else if(c->control_pdu.ETY==200)
  {
      l_n=1;
      Read_Std_Data(stddata,l_n);
      Compute_Std_Total(stddata,l_n);
      Compute_Pass_Fail(stddata,l_n);
      Compute_Average(stddata,l_n);

  }
  if(c->control_pdu.ETY==500 || c->control_pdu.ETY==400 || c->control_pdu.ETY==300 || c->control_pdu.ETY==200 )
  {
      int l_i;
      for(l_i=0;l_i<l_n;l_i++)
      {
          printf("Student Name:%s\n",stddata[l_i].Std_Name);
          printf("Student Number:%d\n",stddata[l_i].Std_No);
          printf("Student marks in subject 1 : %d\n", stddata[l_i].Std_Marks_Data.Marks[0]);
         printf("Student marks in subject 2 : %d\n", stddata[l_i].Std_Marks_Data.Marks[1]);
         printf("Student marks in subject 3 : %d\n", stddata[l_i].Std_Marks_Data.Marks[2]);
         printf("Student marks in subject 4 : %d\n", stddata[l_i].Std_Marks_Data.Marks[3]);
         printf("Student marks in subject 5 : %d\n", stddata[l_i].Std_Marks_Data.Marks[4]);
         printf("Student total marks : %d\n", stddata[l_i].Std_Marks_Data.Std_Score.Std_Total);
         printf("Student percentage : %f\n", stddata[l_i].Std_Marks_Data.Std_Score.Std_Average);
         if(stddata[l_i].Std_Marks_Data.Std_Score.Std_Pass_Fail == pass) {
            printf("Result : Pass\n");
            Compute_Division(stddata, l_n);
			}
      }
  }
}
void Display_Emp_Data(struct AU1_SaPDU *b, struct Emp_data *empdata)
{
    int l_i,l_n;
   if(b->control_pdu.Label==12)
   {
       l_n=1;
       Read_Emp_Details(empdata,l_n);
       Compute_Allowances(empdata,l_n);
       Compute_Deductions(empdata,l_n);
       Compute_Salary(empdata,l_n);
   }
   else if(b->control_pdu.Label==18)
   {
       l_n=5;
       Read_Emp_Details(empdata,l_n);
       Compute_Allowances(empdata,l_n);
       Compute_Deductions(empdata,l_n);
       Compute_Salary(empdata,l_n);
   }
   else if(b->control_pdu.Label==25)
   {
       l_n=10;
       Read_Emp_Details(empdata,l_n);
       Compute_Allowances(empdata,l_n);
       Compute_Deductions(empdata,l_n);
       Compute_Salary(empdata,l_n);
   }
   if(b->control_pdu.Label==12 || b->control_pdu.Label==18 || b->control_pdu.Label==24)
   {
       int l_i;
       for(l_i=0;l_i<l_n;l_i++)
       {
           printf("Total Information of Emp %d is\n",l_i+1);
           printf("Employee Name:%s\n",empdata[l_i].EmpName);
           printf("Employee Number:%d\n",empdata[l_i].EmpNo);
           printf("Employee Basic salary:%d\n",empdata[l_i].EmpBasic.Emp_Basic);
           printf("Employee HRA:%.2f\n",empdata[l_i].EmpBasic.Emp_HRA);
           printf("Employee DA:%.2f\n",empdata[l_i].EmpBasic.Emp_DA);
           printf("Employee CCA:%.2f\n",empdata[l_i].EmpBasic.Emp_CCA);
           printf("Employee PT:%.2f\n",empdata[l_i].EmpBasic.EmpPay.Emp_PT);
           printf("Employee SalaryTax:%.2f\n",empdata[l_i].EmpBasic.EmpPay.Emp_SalaryTax);
           printf("Salary:%f\n",empdata[l_i].EmpBasic.EmpPay.Emp_Salary);

       }
   }
}

int main() {
    struct AU1_SaPDU sapdu;
    struct Emp_data empdata[10];
    struct Std_data stddata[10];

    Compute_Random_Number(&sapdu);
    Display_Emp_Data(&sapdu, empdata);
    print_Std_Data(&sapdu, stddata);

    return 0;
    }
