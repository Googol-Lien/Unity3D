//************************************************************
//* Copyright (c) Googol Lien
//* This file is personal asset of Googol Lien (連谷川),
//* for sharing use by other programmers or 3rd parties only.
//************************************************************
using UnityEngine;
using System;
using System.Collections;

public class TPAudioData : TPObject
{
	public string[] MusicName;
	public AudioClip[] AudioClip;
	public bool bMusic;
	public bool bSound3D;

	AudioSource thisAudio;

	static TPObjectVault Vault = new TPObjectVault();

	class AudioLife
	{
		public GameObject go;
		public AudioSource auds;
		public float time;
	}

	class AudioMute
	{
		public TPObjectSuper TPObj;
		public AudioSource Source;
		public int NewClipIndex = -1;
		public bool bNewLoop;
		public float[] volume;
		public float time;
	}

	IEnumerator Thread_FadeOutUpdateVolume( TPCode t )
	{
		//先等資料設定完成
		yield return null;

		AudioMute am = ( AudioMute ) t.DataObject;

		yield return t.FloatLerp( am.volume, thisAudio.volume, 0.0f, am.time );

		t.DataObject = null;
	}

	IEnumerator Thread_FadeOut( TPCode t )
	{
		//先等資料設定完成
		yield return null;

		AudioMute am = ( AudioMute ) t.DataObject;
		TPCode muv = null;

		am.TPObj.NewThread( Thread_FadeOutUpdateVolume, ref muv );
		muv.DataObject = t.DataObject;

		while ( muv.DataObject != null )
		{
			if ( !am.Source.gameObject.activeSelf ) yield break;
			am.Source.volume = am.volume[ 0 ];
			yield return null;
		}

		am.Source.Stop();

		if ( am.NewClipIndex >= 0 ) Play( am.NewClipIndex, am.bNewLoop );
	}

	IEnumerator Thread_Recycle( TPCode t )
	{
		//先等資料設定完成
		yield return null;

		AudioLife al = ( AudioLife ) t.DataObject;

		yield return t.Idle( al.time );

		al.auds.clip = null;
		Vault.RecycleGameObject( al.go );
	}

	void resetReuse( GameObject go, ref object data )
	{
		data = go.audio;
	}

	GameObject newGameObject( ref object data )
	{
		GameObject go = new GameObject( "_AUDIO" );
		AudioSource source = ( AudioSource ) go.AddComponent( typeof( AudioSource ) );

		source.priority = 255;	//音效預設為最低，以免蓋過音樂
		data = source;

		return go;
	}

	AudioSource PlayNewSource( GameObject go, object aus )
	{
		AudioSource source = ( AudioSource ) aus;

		source.clip = thisAudio.clip;
		source.volume = thisAudio.volume;
		source.pitch = thisAudio.pitch;
		source.loop = thisAudio.loop;
//		source.rolloffFactor = thisAudio.rolloffFactor;

		source.Play();

		if ( !thisAudio.loop )
		{
			TPCode t = null;
			AudioLife al = new AudioLife();

			al.go = go;
			al.auds = source;
			al.time = thisAudio.clip.length / source.pitch;
			NewThread( Thread_Recycle, ref t );
			t.DataObject = al;
		}

		return source;
	}

	AudioSource PlayOneShot()
	{
		object source = null;
		GameObject go = Vault.ReuseGameObject( newGameObject, resetReuse, ref source );

		TPUnity.SetParent( thisTransform.parent, go.transform );

		return PlayNewSource( go, source );
	}

	AudioSource PlayClipAtPoint( Vector3 pos )
	{
		object source = null;
		GameObject go = Vault.ReuseGameObject( newGameObject, resetReuse, ref source );

		go.transform.position = pos;

		return PlayNewSource( go, source );
	}

	public void RecycleAudio( AudioSource ausrc )
	{
		ausrc.clip = null;
		Vault.RecycleGameObject( ausrc.gameObject );
	}

	public void PlayLoaded()
	{
		if ( thisAudio == null ) return;

		thisAudio.loop = false;
		thisAudio.pitch = 1.0f;

		thisAudio.Play();
	}

	public void LoadOnly( int index )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}
	}

	public void Play( int index, bool bLoop )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = bLoop;
		thisAudio.pitch = 1.0f;

		thisAudio.Stop();
		thisAudio.Play();
	}

	public void Play( int index )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = false;
		thisAudio.pitch = 1.0f;

		thisAudio.Play();
	}

	public void Play( int index, float pitch )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = false;

		Pitch( pitch );

		thisAudio.Play();
	}

	public Coroutine FadeOut( float time )
	{
		if ( thisAudio == null ) return null;

		return FadeOut( null, thisAudio, time );
	}

	public Coroutine FadeOut( TPObjectSuper tObj, AudioSource Source, float time )
	{
		if ( Source == null ) return null;
		if ( tObj == null ) tObj = this;

		AudioMute am = new AudioMute();
		TPCode t = null;

		Coroutine cor = tObj.NewThread( Thread_FadeOut, ref t );

		am.TPObj = tObj;
		am.Source = Source;
		am.time = time;
		am.volume = new float[ 1 ] { Source.volume };

		t.DataObject = am;

		return cor;
	}

	public Coroutine FadeOutAndPlay( int newIndex, bool bNewLoop, float time )
	{
		if ( thisAudio == null ) return null;

		return FadeOutAndPlay( newIndex, bNewLoop, null, thisAudio, time );
	}

	public Coroutine FadeOutAndPlay( int newIndex, bool bNewLoop, TPObjectSuper tObj, AudioSource Source, float time )
	{
		if ( Source == null ) return null;
		if ( tObj == null ) tObj = this;

		AudioMute am = new AudioMute();
		TPCode t = null;

		Coroutine cor = tObj.NewThread( Thread_FadeOut, ref t );

		am.TPObj = tObj;
		am.Source = Source;
		am.time = time;
		am.volume = new float[ 1 ] { Source.volume };
		am.NewClipIndex = newIndex;
		am.bNewLoop = bNewLoop;

		t.DataObject = am;

		return cor;
	}

	public AudioSource PlayOne( int index )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = false;
		thisAudio.pitch = 1.0f;

		return PlayOneShot();
	}

	public AudioSource PlayOneVolume( int index, float volRate )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume * volRate;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume * volRate;
		}

		thisAudio.loop = false;
		thisAudio.pitch = 1.0f;

		return PlayOneShot();
	}

	public AudioSource PlayOneVolume( int index, float volRate, bool bLoop )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume * volRate;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume * volRate;
		}

		thisAudio.loop = bLoop;
		thisAudio.pitch = 1.0f;

		return PlayOneShot();
	}

	public AudioSource PlayOne( int index, bool bLoop )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = bLoop;
		thisAudio.pitch = 1.0f;

		return PlayOneShot();
	}

	//播放指定陣列的音效
	public AudioSource PlayOne( AudioClip[] clips, int index, float pitch )
	{
		if ( clips == null || index >= clips.Length ) return null;

		thisAudio.clip = clips[ index ];
		thisAudio.volume = TPBase.SoundVolume;

		thisAudio.loop = false;

		Pitch( pitch );

		return PlayOneShot();
	}

	public AudioSource PlayOne( AudioClip[] clips, int index, float pitch, bool bLoop )
	{
		if ( clips == null || index >= clips.Length ) return null;

		thisAudio.clip = clips[ index ];
		thisAudio.volume = TPBase.SoundVolume;

		thisAudio.loop = bLoop;

		Pitch( pitch );

		return PlayOneShot();
	}

	public AudioSource PlayOne( int index, float pitch )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = false;

		Pitch( pitch );

		return PlayOneShot();
	}

	public AudioSource PlayOne( int index, float pitch, bool bLoop )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = bLoop;

		Pitch( pitch );

		return PlayOneShot();
	}

	public AudioSource PlayAt( Vector3 pos, int index )
	{
		if ( bMusic )
		{
			thisAudio.clip = ( AudioClip ) Resources.Load( MusicName[ index ], typeof( AudioClip ) );
			thisAudio.volume = TPBase.MusicVolume;
		}
		else
		{
			thisAudio.clip = AudioClip[ index ];
			thisAudio.volume = TPBase.SoundVolume;
		}

		thisAudio.loop = false;
		thisAudio.pitch = 1.0f;

		return PlayClipAtPoint( pos );
	}

	public void Stop()
	{
		thisAudio.Stop();
		thisAudio.pitch = 1.0f;
	}

	public void StopAndRecycle( AudioSource aud )
	{
		aud.Stop();
		aud.pitch = 1.0f;
		aud.clip = null;

		if ( aud == thisAudio || aud.gameObject == null ) return;

		Vault.RecycleGameObject( aud.gameObject );
	}

	public void Volume( float vol )
	{
		thisAudio.volume = vol;
	}

	public void Pitch( float p )
	{
		if ( p < 0.5f )
		{
			p = 0.5f;
		}
		else if ( p > 2.0f )
		{
			p = 2.0f;
		}

		thisAudio.pitch = p;
	}

	public override void TPObjectAwake()
	{
		bNoTimeScale = true;	//不受暫停影響
		thisAudio = audio;

		if ( bMusic )
		{
			if ( thisAudio != null ) thisAudio.priority = 0;		//最高
		    TPUnity.SafeDieClear( ref TPBase.MusicObject, this );
		}
		else
		{
			if ( thisAudio != null ) thisAudio.priority = 255;		//最低
		    if ( bSound3D )
		    {
		        TPUnity.SafeDieClear( ref TPBase.SoundObject3D, this );
		    }
		    else
		    {
		        TPUnity.SafeDieClear( ref TPBase.SoundObject, this );
		    }
		}
	}
}