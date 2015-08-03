#!/usr/bin/perl
use File::Glob qw(:globally :nocase);
use Data::Dumper;

$osversion=`sw_vers -productVersion`;
chomp($osversion);
@os_version=split(/\./, $osversion);

$DEBUG=1;
$BASE=sprintf("ports%d%d", $os_version[0], $os_version[1]);
printf("Port base is '%s'\n", $BASE);

$INFOS="/Volumes/munki/pkgsinfo/$BASE";
$PKGS="/Volumes/munki/pkgs/$BASE";

# Check to repack only specific port
($specific) = @ARGV;
if ($specific ne '') {
    print "Importing port $specific\n";
    open(PORTLIST, "/opt/local/bin/port installed 2>/dev/null | grep '^ *$specific ' |") or die "Can't list port $specific\n";
} else {
    open(PORTLIST, "/opt/local/bin/port installed 2>/dev/null | /usr/bin/sed -e '1d' |") or die "Can't list ports\n";
}

open(MANIFEST, ">/tmp/import-manifest" ) || die ("Can't write manifest\n");

while (<PORTLIST>) {
    chomp();
    next unless (/\(active\)$/);
    ($portname, $portmeta)=split();
    ($ver, @variants)=split(/\+/, $portmeta);
    $ver=~s/^\@(.*)_.*/\1/;

    # Fix package names that contain things that Munki mistakes
    #	for version numbers
    ($munkiname=$portname)=~s/-([0-9])/_\1/g;

    # Move on if munki already has this port
    if (-f "$INFOS/$munkiname-$ver.plist") {
	print "INFO: pkg $portname seems to exist in munki already - skipping\n";
	next;
    }

    # Determine dependencies
    @libdeps=(); @rundeps=();
    open(DEP, "port deps $portname 2>/dev/null|");
    while(<DEP>) {
       chomp();
       if (/^Library Dependencies/) {
	    $_=~s/Library Dependencies: //;
	    @libdeps=split(', ');
       }
       if (/^Runtime Dependencies/) {
	    $_=~s/Runtime Dependencies: //;
	    @rundeps=split(', ');
       }
    }

    print "$portname (v $ver)\n";
    $portwork=`port work $portname 2>/dev/null`;
    chomp($portwork);
    if (length($portwork)<2) {
	# No workdir, so no pkg
	$ret=build_port_pkg($portname, \@variants);
	$portwork=`port work $portname 2>/dev/null`;
	chomp($portwork);
	die("No work directory for $portname!\n") if(length($portwork)<2);
    }
    $portpkg=find_port_pkg($portname);
    if(length($portpkg)<2) {
        build_port_pkg($portname, \@variants);
	$portpkg=find_port_pkg($portname);
	die("Can't build pkg for $portname!\n") if (length($portpkg)<2);
    }

    $minver=sprintf("%d.%d.0", $os_version[0], $os_version[1]);
    $maxver=sprintf("%d.%d.99", $os_version[0], $os_version[1]);
    $importargs="--name=$munkiname --pkgvers=$ver --subdirectory $BASE --unattended_install";
    $importargs.=" --minimum_os_version=$minver --maximum_os_version=$maxver";
    foreach $dep (@libdeps, @rundeps) {
	$dep=~s/-([0-9])/_\1/;
	$importargs.=" -r $dep";
    }
    # Find files to take md5 sum for
    @fileslist=find_port_files($portname);
    foreach $portfile (@fileslist) {
    	# Add md5sum for the binary
	$importargs.=" -f $portfile";
    }
    print "DBG- import arguments: $importargs\n" if ($DEBUG);
    if (-f "$PKGS/$portname-$portver.dmg" ) {
	# DMG exists, just re-generate metadata
	$jobout=system("/usr/local/munki/makepkginfo $importargs $portpkg > $INFOS/$portname-$portver.plist");
    } else {
	$jobout=system("/usr/local/munki/munkiimport -n $importargs $portpkg");
    }
    die("munkiimport failed!\n") if ($jobout>0);
    printf MANIFEST "\t\t<string>%s</string>\n", $munkiname;
}

###################################################################################

sub build_port_pkg ($@) {
	my $portname=$_[0];
	my @variants=@{$_[1]};
	print "  Building pkg for $portname...\n";
	$pkgopts=join(' +', @variants);
	print "DBG: port pkg $portname +$pkgopts\n";
	$pbuildstr="sudo port pkg $portname";
	if($pkgopts) {
	    $pbuildstr.=" +$pkgopts";
	}
	print "DBG: build pkg: $pbuildstr\n" if ($DEBUG>0);
	system($pbuildstr);
	return $?;
}

sub find_port_pkg ($) {
    my $portname=$_[0];

    $portwork=`port work $portname 2>/dev/null`;
    chomp($portwork);
    if(length($portwork)<2) {
    	return('');
    }
    @files=glob("$portwork/$portname*pkg");
    $pkgfile='';
    foreach $try (@files) {
        next if ($try=~/-component/);
	$pkgfile=$try;
    }
    if ( -e $pkgfile ) {
	print "Found pkg for $portname\n";
	return($pkgfile);
    } else {
	return('');
    }
}

sub find_port_files ($) {
    my $portname=$_[0];
    my $numlib=0;
    my $numbin=0;
    my $libargs='';
    my $binargs='';
    my @portfiles=();
    open (PF, "port contents $portname|") or die ("Can't list port contents for $portname\n");
    while (my $file=<PF>) {
	chomp($file);
	$file=~s/^ *//;
	next unless (-f $file);
    	if ($file=~/local\/bin\//) {
	    $numbin++;
	    next if($numbin > 2);	# Keep the installs array to reasonable size
	    push (@portfiles, $file);
	}
        if ($file=~/Python.framework.*egg-info/) {
            $numlib++;
            next if($numlib > 2);
            push (@portfiles, $file);
        }
	if ($file=~/local\/lib\/.*dylib/) {
	    $numlib++;
	    next if($numlib > 2);
	    push (@portfiles, $file);
	}
    }
    return (@portfiles);
}
