# CMD�������

## һ����������

> 1���鿴�ļ�����
>
> dir /a ������ʾ��ǰ�ļ��������ļ�
>
> dir /ar������ʾֻ���ļ�
>
> dir /as������ʾϵͳ�ļ�
>
> cd �л�����





> 2��������������������
>
> copy Դ�ļ���/Դ�ļ� Ŀ���ַ������Դ�ļ�/Դ�ļ��и��Ƶ�Ŀ���ַ
>
> xcopy Դ�ļ��� Ŀ���ַ ���� ����xcopy��copy����ǿ�������ʵ�ֶԶ����Ŀ¼���п���
>
> /s ����Ŀ¼����Ŀ¼����������Ŀ¼�� /e ����Ŀ¼����Ŀ¼��������Ŀ¼��
>
> rename Դ�ļ��� Ŀ���ļ���
>
> move Դ�ļ� Ŀ���ַ ���������ƶ�ʱֱ��������



> 3�������޸�����
>
> chkdsk
>
> convert������fat��תΪntfs��convert f: /fs:ntfs //��������F�Ѿ����NTFS
>
> diskpart�����������ģʽ list disk �鿴����
>
> ```bash
> diskpart //��ȥ���̷���ģʽ
> select disk 1 // ѡ�����1
> clean //��ʽ������
> creat partition primary //����������
> format fs=ntfs quick label = ��E:�� //�������
> ```
>
> 
>
> 



> 4�������ҵ��ض��ļ�
>
> ����forѭ��
>
> ```bash
> for /r c:\ %i in (*.txt) do echo %i   //�ҳ�c��������txt��׺���ļ�����ӡ
> ```



> 5����������������
>
> start������������������һ�������̷����ļ����ļ��С���ַ������
>
> call������������� call demo.bat //��ͬһ·������ֱ�ӵ��ã���ͬ·��Ҫ��·����



> 6����ʱ��������ԱȨ�� 
>
> runas 
>
> ```bash
> runas /noprofile /user :mymachine administrator cmd  //����������ԱȨ��
> hostname//�鿴������
> ```



> 7��net use����
>
> �ͶԷ������������ӣ�ӳ�����ݵ�����
>
> ǰ���������Է�����IPC$������ͬһ������
>
> ����IPC���� �ܳ����� IPC���֣���ͨ��ʹ��Windows ϵͳ��Ĭ��������[IPC$](https://baike.baidu.com/item/IPC%24/9482475?fromModule=lemma_inlink)�������ﵽ������������ü��������Ȩ��[����](https://baike.baidu.com/item/����/74119?fromModule=lemma_inlink)������������Ҫ���õ��Ǽ����ʹ���߶�[�������ȫ](https://baike.baidu.com/item/�������ȫ/2950375?fromModule=lemma_inlink)��֪ʶȱ����ͨ�����������������������������ڼ򵥣���˲ŵ��±��ڿ͵��л��ɳˡ�
>
> 1��������������
>
> net use \\\IP\ipc$ "" /user:""           //��仰�������ո�
>
> 2���������ǿ�����
>
> net use \\\IP\ipc$ "�û���" /user:"����"   
>
> 3����ӳ��Ĭ�Ϲ���
>
> net use z:\\\IP\c$ "����" /user:"�û���" //���Է���c��ӳ��Ϊ�Լ���z�̣���������
>
> ����Ѿ���Ŀ�꽨����ipc$,�����ֱ�ӷ���
>
> ```basg
> �����ֱ����IP+�̷�+$���� net use z:\\IP\c$
> ```
>
> 4����ɾ��һ��ipc$����
>
> net use \\ip\ipc$ /del
>
> 5����ɾ������ӳ��
>
> net use c: /del           //ɾ��ӳ���C��



> 8��net ����
>
> net user�����鿴���������û�
>
> net user userNew /add ��������һ���û���ΪuserNew���û�
>
> net user userNew /del ����ɾ��
>
> net user userNew /active ���������û�
>
> net share �����鿴����
>
> net share f=F:/ ��������F��
>
> net share f /delete���� ȡ������
>
> net view \\\������ �����鿴�ض���������
>
> net start + ���񡪡���������
>
> net stop+ ���񡪡��رշ���
>
> netsh interface tcp show global �����鿴tcpȫ�ֲ���
>
> ![image-20230212091646011](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230212091646011.png)
>
> 



> 9��shutdown�ػ�����
>
> shutdown /r -t  120����120s��ػ�
>
> shutdown  /a ����ȡ���ػ��趨

> 10��netstat�����������
>
> netstat -ano  // �鿴������������
>



> 11��wget����������Դ
>
> cmd����������Ҫ���أ�����windows/system32�ļ�����
>
> ����
>
> ```bash
> wget www.baidu.com  //��ȡԶ����ַ��html����
> wget -r ��վ //��ȡ������վ
> ```



> 12��ϵͳ�ļ��޸�
>
> sfc������������ԱȨ��
>
> sfc /scannow ��������ϵͳɨ��
>
> ע��C:\Windows\Logs\CBS �������־�ļ��ܷ�ɾ������־�ļ����ṩ�й��ѻ�������ϵ���ϸ��Ϣ�Ĵ�����־�ļ�������ɾ��
>
> sfc /verifile=�ļ��� ������֤�ļ���������

> 13��nslookup����
>
> nslookup ��Ҫ�����������ϵͳ (DNS) �����ṹ����Ϣ����ѯDNS�ļ�¼����ѯ���������Ƿ����������������ʱ��������������⡣
>
> nslookup ����(www.baidu.com)

## ����������.bat�ļ���д

> ע�����
>
> 1. ELSE �Ӿ���������ͬһ���ϵ� IF 
> 2. /i ��ʾ���Դ�Сд
> 3. set var1=asd //���þֲ�����
> 4. setx PATH ��%path;�ļ���·���� //�������ñ���  setx PATH ��%path;d:�� //�������ñ��� 
> 5. set ������Բ鿴��ǰ�����Ļ�������
> 6. �����ַ�  
> 7. * ����ܵ�����| ���� ����һ����������Ϊ�ڶ����������
>    * ������&��������һ������ִ��ʧ�ܣ������������ִ��
>    * ������&&��������һ������ִ��ʧ�ܣ���������Ҳ����ִ�С���һ���ɹ����ڶ���Ҳ�ɹ�
>    * ������||��������һ������ʧ�ܺ��ִ�еڶ�������
> 8. rem �� :: ��bat�ļ����ע�Ͳ���
> 9. exit����==return
> 10. goto�������ת goto part1          :part1 echo iam ready
> 10. ��ѭ����ĳ������ start ������ %0
> 10. ѭ��������ĳ���� for /l %%i in (1,1,3) do start ������
> 10. sort���� -���� sort demo.txt > 1.txt ������õ�demo.txt�ض���1.txt
> 10. cmd��������ϵͳ����
> 10. forѭ���﷨��for /l %������ in (start,step,end) do command [����]
>
> ```bash
> for /d %i in(*) do echo %i             //��ѯ�����ļ��в���ӡ
> for /r f:\%i in(*.txt) do echo %i                    //�ݹ���������ļ���
> ```
>
>  16.

> ```ba
> @echo off
> if exist F:\  (echo ����Ŷ)  else  (echo  ������Ŷ)
> pause  >nul
> ```
>
> ```bash
> @echo off
> set var1=hello
> set var2=HELLO
> if /i %var1%==%var2%  (echo ���)  else  (echo  �����)
> pause  >nul
> ```
>
> ```bash
> @echo off
> if exist c:\it  (echo ����it�ļ���)  else  (md  c:\it & echo ����it�ļ��гɹ�)
> pause  >nul
> ```
>
> ping 2�����ݰ�����Ƿ���ͨ
>
> ```bash
> @echo off
> set /p var=������IP��ַ:
> ping  -n 2 %var%
> pause >nul
> ```
>
> ���������ļ���
>
> ```bash
> @echo off
> for /l %i in (1,1,10) do md c:\%i.txt
> pause >nul
> 
> 
> 
> 
> ���� ��ȡ�ı��ļ����ݴ����ļ���  /f ��ȡ�ļ�����
> @echo off
> for /f %%i in (c:\demo.txt) do md c:\%%i.txt
> pause >nul
> ```
>
> ����ϵͳ����
>
> ```bash
> @echo off
> echo ����رձ�����!
> echo �������ϵͳ�����ļ������Ե�
> del /f /s /q %systemdrive% *.tmp
> del /f /s /q %windir% prefetch *.* rd /s /q %windir%\temp & md
> %windir%\temp 
> del /f /s /q "userprofile%\ Local Settings\Temp\*.*"
> del /f /s /q %systemdrive% *._mp
> del /f /s /q %windir% *.bak
> del /f /s /q %systemdrive%\ *.log
> del /f /s /q %systemdrive%\ *.gid
> del /f /s /q %systemdrive%\ *.chk
> del /f /s /q %systemdrive%\ *.old
> del /f /s /q %systemdrive%\recycled\*.*
> del /f /s /q %userprofile%\cookies\ *.*
> del /f /s /q %userprofile%\ recent\ *.*
> del /f /s /q "%userprofile%\ Local Settings \Temporary Internet
> Files \*.*"
> del /f /s /q "%userprofile%\recent\*.*del"
> echo ���ϵͳ�������!
> ```
>
> 





## ����Kail��͸

kali������һ������ϵͳ,��ȷ�е�˵����һ������Linux kernel�Ĳ���ϵͳ,��ϵͳ��BackTrack��չ����������������������

> 1��namp��ʹ��
>
> > nmap ��վ���� ����̽��Ŀ����������ǽ״̬
> >
> > nmap -F ��վ������������̽����ٶ�
> >
> > nmap -O ��վ���������鿴�����Ĳ���ϵͳ
> >
> > nmap -sT ��վ���������鿴tcp�˿ڿ������
> >
> > nmap -sU ��վ���������鿴udp�˿ڿ������
> >
> > nmap 192.168.1.100/24����ɨc��0-255����
> >
> > nmap -iL 1.txt�������ı��ж�ȡIP��ַ
