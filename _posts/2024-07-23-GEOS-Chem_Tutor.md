---
layout: posts
title: GEOS-Chem 모델 모의 해보기
author: 도세원, 심창섭, 서정빈
categories: []
tags: [GEOS-Chem]
---



<p style="text-align:right">작성 : 도세원 (한국환경연구원, KEI)<br/> 
& 심창섭 (한국환경연구원, KEI)<br/>& 서정빈 (한국환경연구원, KEI)<br/>2024년 7월 19일</p>


---



## 소 개

[GEOS-Chem은 NASA Global Modeling and Assimilation Office](http://gmao.gsfc.nasa.gov/) 의 Goddard Earth Observing System(GEOS)에서 얻은 기상 입력을 통해 구동되는 대기 화학의 글로벌 3D 모델입니다 .  GEOS-Chem 모델을 통해 광범위한 대기오염물질을 분석할 수 있습니다.

모델의 개발 및 발전을 위하여 [GEOS-Chem Steering Committee](https://geoschem.github.io/steering-committee.html) 와 [Working Group](https://geoschem.github.io/working-groups.html)이 있으며, 이 모델은 은 US NASA Earth Science Division, the Canadian National and Engineering Research Council, and the Nanjing University of Information Sciences and Technology의 지원을 받아 하버드 대학교와 워싱턴 대학교에 있는 [GEOS-Chem 지원 팀](https://geoschem.github.io/support-team.html) 에서 관리합니다 .

자세한 정보와 링크는 [모델 개요 ](https://geoschem.github.io/overview.html) 참조.



GEOS-Chem과 관련된 주요한 소프트웨어는 5가지가 있다.

* **GEOS-Chem Classic** (이번 메뉴얼)
* [GCHP](https://gchp.readthedocs.io/) : 고해상도 버전(GEOS-Chem High Performance)
* [HEMCO](https://hemco.readthedocs.io/) : 배출량 정보
* [GCPy]([gcpy.readthedocs.io]) : Python toolkit
* [WRF-GC](http://wrf.geos-chem.org/) : GEOS-Chem in WRF
* [IMI](https://imi.readthedocs.io/) : Integrated Methane Inversion





## 하드웨어 요구사항

- 운영체제

  **GEOS-Chem Classic은 Unix와 유사한** 운영 체제가 있는 컴퓨터에서만 사용할 수 있다. 여기에는 모든 Linux 플레이버(예: Ubuntu, Fedora, Red-Hat, Rocky Linux, Alma Linux 등)와 BSD Unix(BSD 파생 버전인 MacOS X 포함)가 포함. 귀하의 기관에 컴퓨팅 리소스(예: 많은 코어가 있는 공유 컴퓨터 클러스터, 충분한 [디스크 스토리지](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/disk-space.html#req-hard-disk) 및 [메모리](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/memory.html#req-hard-mem) )가 있는 경우 GEOS-Chem Classic을 실행할 수 있다.

- RAM

  GEOS-Chem Classic 모델은 1개월을 수행하는데 벤치마크 시뮬레이션에서는 약 14GB의 메모리가 사용됨(모든 진단 변수를 저장할 경우). 이러한 이유는 진단 변수들이 모두 저장되기 때문이다. 그러나, 일반적인 상황에서는 모든 진단 변수를 출력하지 않기 때문에 벤치마크 시뮬레이션 보다는 훨씬 적은 메모리를 사용한다.

  - 4º × 5º 해상도 수행 :

    : 8~15GB

  - 2º × 2.5º 해상도 수행

    : 30~40GB 

  - 1º × 1.25º 또는 0.5º × 0.625º  등 고해상도 수행 

    : 최소 30GB 이상 

- 저장공간

  GEOS-Chem Classic에서 사용되는 데이터 형식은 NetCDF가 사용된다([netCDF 가이드](https://geos-chem.readthedocs.io/en/14.2.1/geos-chem-shared-docs/supplemental-guides/netcdf-guide.html#ncguide)). 대략적인 GEOS-Chem Classic 입력 데이터와 출력 데이터를 저장하는데 필요한 공간은 아래와 같다.

  - (입력자료) 

    MERRA-2 데이터 사용시

     :  1º × 1.25º   글로벌   ~30GB

     :  2º × 2.5º   글로벌   ~110GB

     :  0.5º × 0.625º   중앙아시아(AS)   ~115GB

     :  0.5º × 0.625º   유럽(EU)   ~58GB

     :  0.5º × 0.625º   북미(NA)   ~110GB

  - (출력자료) 

    [GEOS-Chem 13.0.0 벤치마크 결과](https://wiki.geos-chem.org/GEOS-Chem_13.0.0#GEOS-Chem_Classic_13.0.0_vs_12.9.0)에 따르면,1개월을 수행한 결과 837MB의 용량이 발생하였다. 이 중에서 진단변수는 ~646MB를 차지하였고, 재시작 파일은 ~191MB를 차지하였다. 공간을 절약하기 위한 자세한 내용은 [GEOS-Chem History 진단](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/history-diag-ref.html#history-diagnostics) 섹션을 참조. 

    (참고) GEOS-Chem 13.0.0부터 코드 수정을 통해 진단 배열이 필요한 출력을 저장하는 데 필요한 충분한 요소로만 차원이 지정되도록 수정되었다. 예를 들어 SpeciesConc_O3 진단만 출력하려는 경우 GEOS-Chem은 관련 배열을 (NX, NY, NZ, 1) 요소(1개 종만 보관하기 때문에 1)로 차원을 지정이 된다. 이를 통해 시뮬레이션에 필요한 메모리 양을 크게 줄일 수 있습니다.

    

    

## 소프트웨어 요구사항

사용하는 소프트웨어에 따라 다양한 상황에 처할 수 있고 설명이 복잡해 지므로, 아래를 참고하길 바란다.

- [컴파일러](https://geos-chem.readthedocs.io/en/latest/getting-started/compilers.html)
  - intel
  - GNU

* [필수 소프트웨어](https://geos-chem.readthedocs.io/en/latest/getting-started/packages-req.html)

  - Git

  - Cmake (3.13 이상)

  - GNU Make (GEOS-Chem 13.0.0 이상에선 필요 없음)

  - NetCDF

- [권장되는 소프트웨어](https://geos-chem.readthedocs.io/en/latest/getting-started/packages-opt.html)

  - GCpy

  - gdb and cgdb

  - ncview

  - nco

  - cdo

  - KPP

  - flex and bison

    

## 1. 모델 및 자료 가져오기

GEOS-Chem 모델을 사용하기 위해서는 모델 소스코를 다운로드가 필요하다.

자세한 설명은  [여기](https://geos-chem.readthedocs.io/en/14.2.1/getting-started/quick-start.html)를 참조.

* GEOS-Chem 모델 다운로드

  ``` bash
  cd ~/MODL/geoschem/
  git clone --recurse-submodules https://github.com/geoschem/GCClassic.git GCClassic
  ```

  참고) 원하는 경우 폴더 이름을 수정할 수 있음

  ```
  git clone --recurse-submodules https://github.com/geoschem/GCClassic.git my_code_dir
  ```



저자의 경우 다운로드 된 GCClassic 버전에 따라 폴더를 분리하기 위하여 추가적인 폴더 정리를 하였음. GCClassic/CHANGELOG.md 파일 안에 기록된 버전에 따라 ~/MODL/geoschem/Code.14.4.1/ 를 생성하여 다운로드 된 GCClassic/ 파일을 이동하였음. (2024.07.19기준 최신버전, v14.4.1)



## 2. 실행 디렉토리 생성

GEOS-Chem을 수행하기 위해 컴파일 할 디렉토리를 성정하여야 한다. 앞으로 여러 실험을 수행할 것을 대비하여 rundirs/ 폴더를 만들고 그 안에 모델을 컴파일 하였다. 



``` bash
mkdir ~/MODL/geoschem/ExtData/
mkdir ~/MODL/geoschem/Code.14.4.1/rundirs
cd ~/MODL/geoschem/Code.14.4.1/GCClassic/run
./createRunDir.sh
```



참고) 현재 사용중인 서버 계정에서 GEOS-Chem 을 처음 수행할 경우 각 계정 안 ./geoschem/config 파일을 생성하기 위하여 몇가지 답변이 필요하다. 아래에 질문사항에 따라 답변을 한다면 큰 어려움이 없다.

``` bash
-----------------------------------------------------------
Enter path for ExtData:  (각자 서버에서 처음 geoschem 모델을 수행시 ./geoschem/config 를 생성하게 된다. [입력자료를 저장한 공간 설정] )
-----------------------------------------------------------   
>>> ~/MODL/geoschem/ExtData/

Initiating User Registration:
You will only need to fill this information out once.
Please respond to all questions.  Registration is required
in order to receive GEOS-Chem support.

-----------------------------------------------------------
What is your name (first and last)?  (./geoschem/comfig 에 저장되는 모델 사용자에 대한 Registration 등록 과정)
What is your email address?
What is the name of your research institution?
What is the name of your principal invesigator?
Please provide the web site for your institution
Please provide your github username 
Where do you plan to run GEOS-Chem?

Successful Registration
-----------------------------------------------------------
```





GEOS-Chem 모델을 수행하기 위한 몇가지 옵션을 선택하여야 한다. 이번 튜토리얼에서는 수행하는 옵션은 아래와 같다. 정상적으로 수행되었다면, 옵션에서 선택한 값들이 사용된 폴더명이 생성될 것이다.

예시) gc_4x5_47L_merra2_CH4

``` bash
-----------------------------------------------------------
Choose simulation type:
-----------------------------------------------------------
   1. Full chemistry
   2. Aerosols only
   3. Carbon
   4. Hg
   5. POPs
   6. Tagged O3
   7. TransportTracers
   8. Trace metals
   9. CH4
  10. CO2
  11. Tagged CO
>>> 9

-----------------------------------------------------------
Choose meteorology source:
-----------------------------------------------------------
  1. MERRA-2 (Recommended)
  2. GEOS-FP
  3. GEOS-IT (Beta release)
  4. GISS ModelE2.1 (GCAP 2.0)
>>> 1

-----------------------------------------------------------
Choose horizontal resolution:
-----------------------------------------------------------
  1. 4.0  x 5.0
  2. 2.0  x 2.5
  3. 0.5  x 0.625
>>> 1

-----------------------------------------------------------
Choose number of levels:
-----------------------------------------------------------
  1. 72 (native)
  2. 47 (reduced)
>>> 2

-----------------------------------------------------------
Enter path where the run directory will be created:
-----------------------------------------------------------
>>> ~/MODL/geoschem/Code.14.4.1/rundirs
Expanding to ~/MODL/geoschem/Code.14.4.1/rundirs

-----------------------------------------------------------
Enter run directory name, or press return to use default:

NOTE: This will be a subfolder of the path you entered above.
-----------------------------------------------------------
>>>
  -- Using default directory name gc_4x5_47L_merra2_CH4
  -- See rundir_vars.txt for summary of default run directory settings
  -- This run directory has been set up to start on 20190101
  -- A restart file for this date has been copied to the Restarts subdirectory
  -- You may add more restart files using format GEOSChem.Restart.YYYYMMDD_HHmmz.nc4
  -- Change simulation start and end dates in configuration file geoschem_config.yml
  -- Default frequency and duration of diagnostics are set to monthly
  -- Modify diagnostic settings in HISTORY.rc and HEMCO_Config.rc

  -- The following sample restart provided for this simulation was not found:
     /home/swdo02/models/geoschem/ExtData/GEOSCHEM_RESTARTS/v2020-02/GEOSChem.Restart.CH4.20190101_0000z.nc4
     You will need to provide this initial restart file or disable
     GC_RESTARTS in HEMCO_Config.rc to initialize your simulation
     with default background species concentrations.

-----------------------------------------------------------
Do you want to track run directory changes with git? (y/n)
-----------------------------------------------------------
>>> n


Created  ~/MODL/geoschem/Code.14.4.1/rundirs/gc_4x5_47L_merra2_CH4
```





## 3. 빌드 구성 및 컴파일

GEOS-Chem은 실행 디렉토리 또는 시스템의 다른 곳에서 빌드할 수 있다. 그러나, 실행 디렉토리에서 GEOS-Chem을 빌드하는 것을 추천한다.  실행 디렉토리에서 빌드를 수행할 경우 모든 빌드 파일이 모델을 실행할 위치와 가까운 곳에 보관되므로 편리하게 사용할 수 있다.

``` bash
cd ~/MODL/geoschem/Code.14.4.1/rundirs/gc_4x5_47L_merra2_CH4/build
cmake ../CodeDir -DRUNDIR=..
make -j
make install
```

위 과정을 따라서 컴파일에 성공하였다면 /build/bin/gcclassic 파일이 생성된다.







## 모든 실험을 위한 기본 사항

GEOS-Chem 모델을 수행하기 위한 빌드 및 컴파일이 끝났다면, 이제는 모델 수행에 필요한 옵션을 설정하여야 한다. 모델을 수행하기 전 아래의 파일에 대한 설정을 검토하여야 한다. [옵션 가이즈 참조](https://geos-chem.readthedocs.io/en/14.2.1/geos-chem-shared-docs/supplemental-guides/customize-guide.html#customguide)



* README : 코드 및 모델 설정 및 실행 방법에 대한 유용한 정보가 들어 있음

*  [geoschem_config.yml](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/geoschem-config.html#cfg-gc-yml) : 실험에 대한 각종 설정(시작 및 종료 시간, 켜거나 끌 작업 등) 수정.

* [HISTORY.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/history.html#cfg-hist) : GEOS-Chem 진단 변수에 대한 설정.

* [HEMCO_Diagn.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/hemco-diagn.html#cfg-hco-diagn) : HEMCO 통해 배출량에 대한 설정.

* [HEMCO_Config.rc](https://geos-chem.readthedocs.io/en/14.2.1/gcclassic-user-guide/hemco-config.html#cfg-hco-cfg) : HEMCO 통해 저장공간(~/MODL/geoschem/ExtData/) 에서 어떤 배출량 인벤토리를 읽어들일지 설정.

모든 실험에 대한 옵션 설정이 끝났다면, GEOS-Chem을 실행하게 된다.





## 4. geoschem_config.yml

이 파일은 모델을 수행하는데 필요한 옵션을 설정하는 텍스트 기반의 파일이다(GEOS-Chem 14.0.0부터 input.geos 대체됨).

* 4.1 모델 실험기간 설정

  ```
  ### geoschem_config.yml: GEOS-Chem Runtime configuration options.
  ### Customized for simulations using the CH4 mechanism.
  
  #============================================================================
  # Simulation settings
  #============================================================================
  simulation:                                                             # GEOS-Chem 시뮬레이션 유형 설정, 'fullchem','aerosol'
    name: CH4                                                             # 'carbon','CH4','CO2','Hg','POPs','TransprotTracers'
    start_date: [20190101, 000000]                                        # 시작시간 설정 [YYYYMMDD, hhmmss]
    end_date: [20190201, 000000]                                          # 끝시간 설정 [YYYYMMDD, hhmmss]
    root_data_dir: /home/swdo/MODL/geoschem/gcgrid/data/ExtData           # root 경로, HEMCO 등 입력자료가 저장된 위치
    met_field: MERRA2                                                     # GEOS-Chem 기상입력자료 : 'MERRA2','GEOS-FP','FCPA2'
    species_database_file: ./species_database.yml                         # GEOS-Chem Species Database 파일 경로
    species_metadata_output_file: OutputDir/geoschem_species_metadata.yml #  
    verbose:                                                              # LOG 출력을 설정
      activate: false                                                     # activate : log 정보를 활성화(true), 비활성화(false)
      on_cores: root       # Allowed values: root all                     # on_cores : 어떤 노드('root','all')에 대한 정보 출력 
    use_gcclassic_timers: false                                           # GEOS-Chem 수행에 걸리는 시간 출력할지 설정
  
  ```

* 4.2 그리드 설정

  ```
  #============================================================================
  # Grid settings
  #============================================================================
  grid:
    resolution: 4.0x5.0                                                   # 수평해상도 설정 : 4º×5º, 2.0º×2.5º, 0.5º×0.625º
    number_of_levels: 72                                                  # 수직해상도 설정 : 72, 47, 40(GCAP2) 
    longitude:                                                            
      range: [-180.0, 180.0]                                              # 위도의 범위 설정 [최소, 최대]
      center_at_180: true                                                 # true 일 경우, 
    latitude:                                                             
      range: [-90.0, 90.0]                                                # 위도의 범위 설정 [최소, 최대]
      half_size_polar_boxes: true                                         # ture 일 경우,
    nested_grid_simulation:                                               # nesting 그리드 사용에 대한 옵션 
      activate: false                                                     # true : nesting, false : 단일 grid
      buffer_zone_NSEW: [0, 0, 0, 0]                                      # nesting grid에서 buffer zone 영역 설정 (3,3,3,3) 추천
      
  ```

- 4.3 모델 적분 시간 설정

  ```
  #============================================================================
  # Timesteps settings
  #============================================================================
  timesteps:                                                              # 모델의 적분시간 간격 설정
    transport_timestep_in_s: 600                                          # 대기수송모델 계산 간격 전역(600), nesting(300)
    chemistry_timestep_in_s: 1200                                         # 화학반응 및 배출 간격: (추천) 전역(1200), nesting(600)
    # radiation_timestep_in_s                                             # fullchem 사용시 RRTMG 복사모델 계산 간격
  
  ```

- 4.4 변수 입력 설정

  ```
  #============================================================================
  # Settings for GEOS-Chem operations
  #============================================================================
  operations:
    chemistry:                                                           
      activate: true                                                     # GEOS-Chem 에서 화학반응 활성화(true), 비활성화(false)
    convection:                                                          
      activate: true                                                     # GEOS-Chem 에서 구름 대류 활성화(true), 비활성화(false)
    pbl_mixing:                                                           
       activate: true                                                    # GEOS-Chem 에서 PBL 혼합 활성화(true), 비활성화(false)
       use_non_local_pbl: true                                           # non-local PBL mixing scheme (VDIFF) 사용 유무
  
    transport:
      gcclassic_tpcore:                 # GEOS-Chem Classic only         # 대류를 통한 물질 이동 제어 옵션
        activate: true                  # GEOS-Chem Classic only         # 활성화(true), 비활성화(false)
        fill_negative_values: true      # GEOS-Chem Classic only         # true : 음수일 경우 농도=0, false : 농도변화 없음
        iord_jord_kord: [3, 3, 7]       # GEOS-Chem Classic only         # 대류옵션 지정 
                                                                         # 1. 1st order upstream scheme, 2.2nd order van Lee
                                                                         # 3. Monotonic PPM, 4.Semi-monotonic PPM
                                                                         # 5.Positive-definite PPM, 6.Un-constrained PPM
                                                                         # 7.Huynh/Van Leer/Lin full monotonicity constrain
                                                                         # 권장옵션 [3, 3, 7]
      transported_species:
        - CH4                                                            # 대류를 통해 이동되는 물질
  
  #============================================================================
  # Options for CH4
  #============================================================================
  CH4_simulation_options:
  
    use_observational_operators:                                         # CH4 관측에 관한 옵션
      AIRS: false                                                        # 비행기 관측 사용 유무, [defalte, false]
      GOSAT: false                                                       # GOSAT 관측 사용 유무, [defalte, false]
      TCCON: false                                                       # TCCON 관측 사용 유무, [defalte, false]
  
    analytical_inversion:
      perturb_CH4_boundary_conditions: false                             # nesting 사용시 경계조건에서 교란 활성화(true)
      CH4_boundary_condition_ppb_increase_NSEW: [0.0, 0.0, 0.0, 0.0]     # nesting 사용시 경계조건에서 교란량(ppbv) 설정
  
  
  ```



- 4.5 추가적인 변수 입력 설정

  ```
  #============================================================================
  # Settings for diagnostics (other than HISTORY and HEMCO)
  #============================================================================
  extra_diagnostics:
  
    obspack:                                                             
      activate: false                                                    # ObsPack 자료를 추가적으로 사용할 경우(true)
      quiet_logfile_output: false                                        # log 파일 생성시 (true)
      input_file: ./obspack_co2_1_OCO2MIP_2018-11-28.YYYYMMDD.nc         # ObsPack 파일의 경로 설정
      output_file: ./OutputDir/GEOSChem.ObsPack.YYYYMMDD_hhmmz.nc4       # 
      output_species:                                                    # 출력 파일에 사용되는 변수
        - CH4
  
    planeflight:
      activate: false                                                    # 비행기 관측 자료를 추가적으로 사용할 경우 (true) 
      flight_track_file: Planeflight.dat.YYYYMMDD                        # 비행기 관측 자료 경로 설정
      output_file: plane.log.YYYYMMDD
  
  ```













## 4. geoschem_config.yml

GEOS-Chem 모델을 수행하기 위한 빌드 및 컴파일이 끝났다면, 이제는 모델 수행에 필요한 옵션을 설정하여야 한다. 모델을 수행하기 전 아래의 파일에 대한 설정을 검토하여야 한다. [옵션 가이즈 참조](https://geos-chem.readthedocs.io/en/14.2.1/geos-chem-shared-docs/supplemental-guides/customize-guide.html#customguide)











```
cd ~/models/WRF/V4.1/WPS
vi namelist.wps
   &share
    wrf_core = 'ARW',              # 역학코어 설정. 연구용:'ARW' or 현업용:'NMM'
    max_dom = 2,                   # nesting grid를 포함한 영역의 갯수. 
    io_form_geogrid = 2,           # geogrid 데이터 형식. 1=binary, 2=netcdf, 3=GRIB1
   /
   &geogrid
     parent_id         =   1,   1, # mother domain의 id. 첫 영역의 1은 자기 자신을 의미함
     parent_grid_ratio =   1,   3, # nested domain의 해상도. 3은 3배를 의미함
     i_parent_start    =   1,  92, # domain의 x 축 시작점 (격자점)
     j_parent_start    =   1,  53, # domain의 y 축 시작점 (격자점)
     e_we              = 341, 331, # x 축 방향의 격자 갯수
     e_sn              = 341, 331, # y 축 방향의 격자 갯수
     geog_data_res = '5m','2m',    # 사용할 지형 데이터의 해상도 설정 (참조: geogrid/GEOGRID.TBL)
     dx = 18000,                   # 최상위 domain의 x 축 방향 모형 해상도(meter)
     dy = 18000,                   # 최상위 domain의 y 축 방향 모형 해상도(meter)
     map_proj = 'lambert',         # 지도 투영 방법. 중위도는 'lambert', 저위도는 'mercater' 사용
     ref_lat   =  35.5,            # 최상위 domain의 중심 위도
     ref_lon   = 144.4,            # 최상위 domain의 중심 경도
     truelat1  =  20.0,            # 지도투영에 의해 발생하는 위도 왜곡에서 지표면과 교차되는 지점1
     truelat2  =  40.0,            # 지도투영에 의해 발생하는 위도 왜곡에서 지표면과 교차되는 지점2
     stand_lon = 144.4,            # 투영에서 수직으로 보여질 경도 (보통은 ref_lon과 같음)
     geog_data_path = '../../Data/geog/WPS_GEOG' # 지형 데이터가 있는 곳의 경로
   /
```









## . GEOS-Chem Classic 실행

GEOS-Chem은 실행 디렉토리 또는 시스템의 다른 곳에서 빌드할 수 있다. 그러나, 실행 디렉토리에서 GEOS-Chem을 빌드하는 것을 추천한다.  실행 디렉토리에서 빌드를 수행할 경우 모든 빌드 파일이 모델을 실행할 위치와 가까운 곳에 보관되므로 편리하게 사용할 수 있다.

``` bash
cd ~/MODL/geoschem/Code.14.4.1/rundirs/gc_4x5_47L_merra2_CH4/build
cmake ../CodeDir -DRUNDIR=..
make -j
make install
```

위 과정을 따라서 컴파일에 성공하였다면 /build/bin/gcclassic 파일이 생성된다.













WRF를 사용하기 위해서는 모델 소스코드와 전처리(WPS), 그리고 지형(geog)자료가 필요합니다.
자세한 설명은 [여기](http://www2.mmm.ucar.edu/wrf/users/download/get_sources.html) 를 참조하세요.

홈페이지에서는 모델과 전처리, 지형자료를 모두 독립적으로 제공하고 있으며, 이상 실험의 경우는 "WRF 모델"만 받으면 됩니다.
가져오는 방법은 ftp로 직접 받아오는 방법과 git을 사용하여 최신 버전을 받는 방법이 있습니다.

선택 1) FTP로 직접 받는 방법 : (버전 V4.0, 2018년 6월 8일 배포)

* WRF 모델 :

  ``` bash
  cd ~/models/WRF/V4.0
  wget http://www2.mmm.ucar.edu/wrf/src/WRFV4.0.TAR.gz
  tar xvzf WRFV4.0.TAR.gz
  ```

* WPS 전처리 :

  ```bash
  cd ~/models/WRF/V4.0
  wget http://www2.mmm.ucar.edu/wrf/src/WPSV4.0.TAR.gz
  tar xvzf WPSV4.0.TAR.gz
  ```

선택 2) Git을 사용하여 최신 버전을 받는 방법 : (현재 버전 V4.1, 2019년 4월 12일 배포)

* WRF 모델 :

  ```bash
  cd ~/models/WRF/V4.1
  git clone https://github.com/wrf-model/WRF
  ```

* WPS 전처리 :

  ```bash
  cd ~/models/WRF/V4.1
  git clone https://github.com/wrf-model/WPS
  ```

지형 자료는 FTP로 직접 받아서 풀어 줍니다.

* WPS Geography data :

  * 고해상도 자료 : 일반적인 용도

    ```bash
    cd ~/models/WRF/Data/geog
    wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_high_res_mandatory.tar.gz
    tar xvzf geog_high_res_mandatory.tar.gz
    ```

  * 저해상도 자료 : 모델 테스트 또는 교육용

    ```bash
    cd ~/models/WRF/Data/geog
    wget http://www2.mmm.ucar.edu/wrf/src/wps_files/geog_low_res_mandatory.tar.gz
    tar xvzf geog_low_res_mandatory.tar.gz
    ```

  

## WRF 모델 디렉토리 구조 설명

WRF 소스코드에 대한 느낌을 알기 위해 아래 사항을 한번 훑어 보시기 바랍니다.

* 모델 소스코드 디렉토리
  * dyn_en/ : ARW 모델의 역학부분
  * dyn_nmm/ : NMM 모델의 역학부분. 더 이상 개선이나 지원을 멈춘 상태임
  * dyn_exp/ : 'toy' 역학 부분
  * external/ : 입출력과 시간 조절, MPI 병렬화 등을 지원
  * frame/ : WRF 프레임워크
  * inc/ : include 디렉토리
  * main/ : wrf.F가 포함된 메인루틴. 컴파일 후 모든 실행화일이 여기에 저장됨
  * phys/ : 모델의 물리부분
  * share/ : WRF의 공유와 입출력 담당
  * tools/ : 개발자용 도구
* 스크립트들
  * clean : 실행화일이나 컴파일 시 생성된 화일들 삭제
  * compile : WRF 코드 컴파일 스크립트
  * configure : 모델 구성용 configure.wrf 를 생성하는 스크립트
* 기타
  * Makefile : 최상위의 makefile
  * Registry : WRF Registry 파일
  * arch/ : 컴파일 옵션이 수집되는 디렉토리
  * run/ : WRF를 실행할 수 있는 디렉토리
  * test/ : 여러 테스트용 실험들이 저장되어 있으며 WRF에서 사용함 

---

---



## 2. WRF 모델의 구성



```bash
export NETCDF=/opt/netcdf/intel-18.0/netcdf-4.1.3
```

netCDF는 지정한 포르란 컴파일러와 동일한 버전으로 생성된 라이브러리여야 합니다.
그리고 netCDF v4.x 부터는 C와 Fortran 라이브러리가 독립적으로 구성되어 있으며 포트란 버전인 libnetcdff.a 가 있는지 확인하세요. (이름 뒤에 f 가 두 개 입니다.)
netCDF가 설치되어있지 않다면 다음과 같이 설치합니다. 버전 4.1.3 이후부터는 C와 Fortran의 라이브러리가 나누어져 있어서 설치가 복잡하므로 설명을 간단히 하기 위하여 v4.1.3을 기준으로 설명합니다. 병렬 시스템의 모든 서버에서 사용 가능하도록 /opt에 설치하도록 합니다. 설치에 대한 자세한 사항은 [여기](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php#STEP2) 를 참조하세요.

```bash
cd ~/local/src
wget http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/netcdf-4.1.3.tar.gz
tar xvzf netcdf-4.1.3.tar.gz
export NETCDF=/opt/netcdf/intel-18.0.4/netcdf-4.1.3
export CC=icc
export CXX=icc
export FC=ifort
export FCFLAGS="-w -O3 -ip -fp-model precise"
export F77=ifort
export FFLAGS="-w -O3 -ip -fp-model precise"
cd netcdf-4.1.3
./configure --prefix=${NETCDF} --disable-dap --disable-netcdf-4 --disable-shared
make && make install
export NETCDF_classic=1
```

**참고**) netCDF v4.x에서는 HDF5와 연동한 빠른 압축기능 및 group 기능 등을 지원합니다. 이를 위해서는 netCDF 설치에 앞서서 HDF5 라이브러리와 Zlib 또는 SZlib 의 설치가 필요합니다. 이에 대한 상세한 설치 방법은 [NetCDF V4 설치하기](http://blog.dhkim.info/NetCDF4/) 를 참조하세요. 
이러한 기능을 사용하지 않고 간단하게 설치하여 사용하고자 한다면 위와 같이  "—disable-netcdf-4" 옵션을 사용하여 컴파일 하고, WRF가 이를 알 수 있도록 "NETCDF_classic" 환경변수를 1로 설정합니다.. 

Configure를 사용하여 모델을 구성합니다.

```bash
cd ~/models/WRF/V4.1/WRF
./configure
> Enter selection [1-63] : 15
> Compile for nesting? (1=basic, 2=preset moves, 3=vortex following) [default 1]: 1
```

> (serial) : 1개의 CPU 또는 core를 사용할 경우
> (smpar) : Shared-memory 즉, OpenMP를 사용할 경우
> (dmpar) : Distributed-memory 즉, MPI를 사용할 경우
> (dm+sm) : OpenMP와 MPI를 동시에 사용할 경우

병렬화와 관련해서는 보통은 성능이 가장 좋은 분산메모리 병렬화만을 사용하므로 "dmpar" 를 선택합니다. dmpar 는 distrubuted memory parallelization을 뜻합니다. 여기에서는 intel fortran compiler를 사용하므로 15. (dmpar) 를 선택하였습니다.
병렬화에 익숙하지 않다면 싱글(serial) 모드를 선택하여 모델에 친숙해 진 후 병렬화를 시도하도록 하세요.

Nesting 관련 옵션은 보통 "1=basic"을 선택합니다.
"2=preset moves"는 namelist 에서 nesting grid의 이동을 직접 지정할 경우 사용합니다.
"3=vortex-following"는 태풍이나 허리케인의 와류 중심을 따라서 nesting grid를 자동으로 이동할 경우 사용합니다. 이에 대한 자세한 설정 방법은 [여기](http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3/users_guide_chap5.htm#movenest) 를 참조하세요.

configure.wrf 가 생성되면 파일 내용 중에 다음을 확인하여 설정과 다르면 알맞게 수정하여 재저장합니다.

```bas
DEP_LIB_PATH    = -L/opt/netcdf/intel-18.0.4/netcdf-4.1.3/lib
SFC             =       ifort
SCC             =       icc
CCOMP           =       icc
DM_FC           =       mpif90 -f90=$(SFC)
DM_CC           =       mpicc -cc=$(SCC)
```



## 3. 실제 사례 실험에 대한 WRF 컴파일

이상 실험에 대한 WRF 컴파일은 [여기](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/Compile/arw_compile3_ideal.php) 를 참고하세요.
일부 컴퓨터에서는 컴파일 하기 전에 "export WRF_EM_CORE=1"를 설정해야 하는 경우가 있으며 잘 모르겠으면 그냥 설정해 주세요.

```bash
export WRF_EM_CORE=1
./compile em_real >& log.compile
```

compile에 사용하는 옵션은 "./compile" 명령을 옵션 없이 실행하면 상세한 설명을 표시해 줍니다. 지금 ./compile 라고만 쳐보세요.

여기서는 실제 사례 실험을 할 것이므로 "em_real" 옵션을 선택하였습니다. 컴파일 시에 표시되는 표준 메세지와 오류 메세지는 ">&" 기능으로 log.compile 이라는 파일에 저장됩니다. 컴파일 후의 성공 여부는 log.compile의 내용을 보면 되며 문제가 있을 경우 이 파일이 문제 해결에 도움이 될 것 입니다.
컴파일은 몇(십)분 정도 걸리는데, 좀 더 빨리 컴파일 하고 싶다면 다음과 같이 -j 옵션으로 병렬 컴파일을 할 수 있습니다.

```bash
./compile -j 8 em_real >& log.compile
```

컴파일이 성공하면 "WRF/main/" 디렉토리에 다음과 같은 실행화일들이 생성됩니다. 실행화일의 크기가 0이 아닌지 확인하세요.
이 실행화일들은 main/ 에서 run/ 및 test/em_real/ 디렉토리에 각각 링크 됩니다. 이 디렉토리 들 중의 하나에서 코드를 실행하면 됩니다.

* ndown.exe : one-way nesting에 사용
* tc.exe : 태풍이나 허리케인 같은 열대성 저기압의 추가 또는 제거에 필요한 TC Bogusing에 사용
* real.exe : 실제 사례에 대한 WRF 초기화
* wrf.exe : WRF 모델의 실제 적분 

컴파일에 문제가 발생하여 실행화일이 제대로 만들어지지 않았다면, log.compile 내에 "Error"을 찾아서 원인을 파악하세요. 원인이 파악되었으면 "./clean -a; ./configure" 를 실행하고 configure.wrf 재 확인 후 다시 컴파일을 시도해 보세요.

```bash
./clean -a # 새로 생성된 파일들을 모두 삭제
./configure # 생성된 configure.wrf의 내용 확인
./compile -j 8 em_real >& log.compile
```

---



WRF가 성공적으로 컴파일 되면 이제 WPS를 컴파일할 차례 입니다. 이 과정은 실제 사례의 실험인 경우게 해당됩니다.

## WPS (WRF 전처리 시스템) : 디렉토리 구조 설명

* README : 코드 및 모델 설정 및 실행 방법에 대한 유용한 정보가 들어 있음
* 소스 코드 디렉토리 :
  * geogrid/ : 지형 격자와 같은 정적 데이터 생성 관련 디렉토리
  * metgrid/ : WRF의 입력자료 생성 관련 디렉토리
  * ungrib/ : GRIB 데이터 압축 해제 관련 디렉토리
  * util/ : 유틸리티 디렉토리
* 스크립트들 :
  * clean : 새로 생성된 파일 또는 실행화일들의 삭제
  * compile : WPS 코드 컴파일
  * configure : WPS 컴파일 환경 구성. configure.wps가 생성됨
  * link_grib.csh : GRIB 파일을 WPS 디렉토리로 링크
* 기타 :
  * arch/ : 컴파일 옵션들이 수집되는 곳
  * namelist.wps : "geogrid.exe"와 "ungrib.exe", "metgrid.exe" 에서 사용하는 WPS namelist.
    "./configure" 명령으로 생성됨
  * namelist.wps-all_options : 참조를 위한 모든 namelist 옵션이 들어가 있는 파일



## 4. WPS의 구성 및 Grib2 관련 라이브러리들의 설치

```bash
echo $NETCDF # netCDF 관련 환경변수가 설정되어 있지 않다면 알맞게 설정해 준다.
cd ~/models/WRF/V4.1/WPS
./configure
> Enter selection [1-36] : 17 # 17. Linux x86_64, Intel compiler (serial)
# "configure.wps" 가 정상적으로 생성되었는지 확인
```

WPS는 WRF의 입력자료를 생성하는 전처리 과정이므로 아주 큰 영역의 실험이 아닌 경우에는 일반적으로 GRIB2를 처리할 수 있는 싱글(serial)을 사용하는 것을 추천합니다. 여기에서는 Intel Compiler를 사용하는 17번을 선택하였습니다.

또한, GRIB2 자료를 사용해야 하므로 관련된 라이브러리(JasPer, libPNG, Zlib)를 확인하고 설치되어 있지 않다면 다음과 같이 설치합니다. 모든 라이브러리는 WRF 모델에서 사용한 컴파일러와 동일한 것을 사용하는 것을 추천합니다. 좀 더 자세한 설명은 [여기](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php#STEP2) 의 지침을 따르도록 하세요.

GRIB2에서 사용하는 JasPer와 libPNG, Zlib 등은 일반적인 라이브러리이므로 여기에서는 사용자 home의 local 디렉토리(${HOME}/local)에 설치하는 것으로 설명합니다.
설치가 정상적으로 끝나면 위에서 생성한 "configure.wps"의 내용 중에서 COMPRESSION_LIBS와 COMPRESSION_INC의 환경변수를 여기서 설치한 디렉토리로 수정해 주어야 합니다. 



## 10. 모의 결과 분석

사례 실험 모의가 끝나면 다음과 같은 결과 파일이 생성됩니다.

> wrfout_d01_2018-08-18_12:00:00, ...

이 결과 파일들은 netCDF 형식으로 만들어진 것이므로 [ncview](http://meteora.ucsd.edu/~pierce/ncview_home_page.html) 등으로 간단히 확인할 수 있습니다.

WRF의 홈페이지에서 제공하는 [WRF Post-Processing Software](http://www2.mmm.ucar.edu/wrf/users/download/get_sources.html#post_processing) 에 편리한 도구들이 있으니 참고하세요.
여기에서는 Python을 이용하여 자료를 분석하고자 하며, 
상세한 내용은 추후 다른 지면으로 통하여 제공하도록 하겠습니다. 

<center>수고하셨습니다! ^-^.</center>



<p style="text-align:right">작성 : 김동훈 (<a href="http://blog.dhkim.info">http://blog.dhkim.info</a>, 인하대학교)<br/> 
& 문일주 (제주대학교)<br/>2019년 4월 29일</p>

---

<center>- END -</center>

---

 

 

 
