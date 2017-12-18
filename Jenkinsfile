node {
	stage('checkout'){
		checkout scm
		//git branch: 'jenkins2-course', url: 'https://github.com/tulasidamarla/solitaire-systemjs-course.git'
	}
	
	stage('build'){
		// pull dependencies from npm
		bat 'npm install'
	}

	stage('stash'){
		// stash code & dependencies to expedite subsequent testing
		// and ensure same code & dependencies are used throughout the pipeline
		// stash is a temporary archive
		stash name: 'everything', 
			  excludes: 'test-results/**', 
			  includes: '**'
	}
	
	stage('test'){
		// test with PhantomJS for "fast" "generic" results
		bat 'npm run test-single-run -- --browsers PhantomJS'
	}
	
	stage('archive'){
		// archive karma test results (karma is configured to export junit xml files)
		step([$class: 'JUnitResultArchiver', 
			  testResults: 'test-results/**/test-results.xml'])
	}
	
	stage('notification'){
	    notify('Build Success')
	}
}

// demoing a second agent
node('windows') {
    //sh 'ls'
	bat 'dir'

    //sh 'rm -rf *'
	bat 'del /S /Q *'

    unstash 'everything'

    //sh 'ls'
	bat 'dir'
}

stage('Browser Testing'){
    parallel chrome: {
        runTests("Chrome")
    }, firefox: {
        runTests("Firefox")
    }
}    

def runTests(browser) {
    node {
        //sh 'rm -rf *'
		bat 'del /S /Q *'
		
        unstash 'everything'

        //sh "npm run test-single-run -- --browsers ${browser}"
		bat "npm run test-single-run -- --browsers ${browser}"
        
		step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }
}

node{
    notify('Deploy to Staging?')
}

input 'Deploy to Staging?'

stage name: 'Deploy to staging', concurrency: 1

node{
    bat 'npm start '
    notify('Solitaire Deployed')
}


def notify(status){
    emailext (
      to: "tulasi.damarla@emc.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
